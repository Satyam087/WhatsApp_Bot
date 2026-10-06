# Engineering architecture — reliability, safety and quality

Researched 2026-09-24. Supersedes the stack sections of `PLAN_V0.md`.
Context: standalone project, **3 engineers, no 1-week deadline**. The real deadline is the June 2027
counselling peak, which gives us ~8 months. We should use them.

---

## 0. Requirements, translated into engineering

| Your requirement | What it actually means | Mechanism |
|---|---|---|
| "should not hallucinate" | No number, fee, date or claim that isn't in our sourced data | 3 data layers + **grounding guard** + claim-level entailment (§4) |
| "should not answer out of scope" | Refuses non-college topics; refuses colleges we haven't verified | **Input rails** + scope contract (§5) |
| "should remain polite" | Never condescending, never pushy, never cold to a distressed kid | Tone rails + output classifier + persona few-shots (§7) |
| "accurate" | Arithmetic exact, facts sourced and dated | Deterministic compute + SQL, never the model (§3) |
| "properly trained" | Consistent Hinglish senior register | Prompt → few-shot → **LoRA on our own transcripts** (§8) |
| "handle all major scenarios" | Doesn't fall over on the 40 real situations | Scenario matrix + eval suite (§9, §11) |
| "should reason out" | Defensible recommendations, not vibes | Decomposition + explicit state machine (§6) |
| "engaging" | Student actually finishes the conversation | Message design + persona (§7) |
| "handle all edge cases" | The long tail | Edge-case taxonomy (§10) |

---

## 1. The core principle: defence in depth, LLM in the middle

The LLM is the least trustworthy component in the system, so it gets the smallest job and the most
supervision. Everything above and below it is deterministic.

```
  INBOUND
     │
  ┌──▼────────────────────────────────────────────┐
  │ RAIL 1 · INPUT      safety · scope · language │  can abort everything
  └──┬────────────────────────────────────────────┘
  ┌──▼────────────────────────────────────────────┐
  │ RAIL 2 · DIALOG     which stage, what's next  │  deterministic state machine
  └──┬────────────────────────────────────────────┘
  ┌──▼────────────────────────────────────────────┐
  │ RAIL 3 · RETRIEVAL  compute · SQL · pgvector  │  builds the FACT BLOCK
  └──┬────────────────────────────────────────────┘
  ┌──▼────────────────────────────────────────────┐
  │        ✳ LLM — phrase the fact block only     │  ← the only non-deterministic step
  └──┬────────────────────────────────────────────┘
  ┌──▼────────────────────────────────────────────┐
  │ RAIL 4 · OUTPUT     grounding · tone · policy │  blocks, doesn't warn
  └──┬────────────────────────────────────────────┘
  ┌──▼────────────────────────────────────────────┐
  │ RAIL 5 · DELIVERY   24h window · length · log │
  └──┬────────────────────────────────────────────┘
  OUTBOUND
```

The five-rail structure is borrowed from **NVIDIA NeMo Guardrails** (input / dialog / retrieval /
execution / output). We take the architecture, not the framework — see §2.

---

## 2. Stack decisions

### Model layer: **Vercel AI SDK** — not an agent framework

| Option | Verdict |
|---|---|
| **Vercel AI SDK** | ✅ **Chosen.** 12.6M weekly downloads, broadest adoption. `generateObject` + Zod gives constrained decoding, which *is* our `Decision` port. Native to Vercel AI Gateway → `typesafe-ai/jev` available for the v1 shadow eval. |
| Mastra | Good, TS-native, YC W25 — but ships opinions on workflows, memory and RAG that we'd spend time overriding. Our state machine *is* the product. |
| LangGraph.js | Trails Python by 4–8 weeks per release and "inherits Python idioms that feel wrong in a Next.js codebase". Graph abstraction is overkill for 7 mostly-linear stages. |

**We are deliberately not building an agent.** Agentic autonomy is the wrong shape here — we want the
model to have *less* freedom, not more. The AI SDK is used as a model-access library, and the
conversation logic is ours.

### Guardrails: **own rails in TypeScript**, informed by NeMo's architecture

| Option | Verdict |
|---|---|
| NeMo Guardrails | **Python-only.** Colang DSL + 5-stage pipeline is genuinely the best design — but adopting it means running a Python sidecar for three engineers to maintain. **We take the 5-rail architecture, not the runtime.** |
| Guardrails AI | Python + JS, 50+ validator hub, 50–200ms/validation. JS support is secondary. |
| **Own rails** | ✅ **Chosen** for domain logic |

The decisive argument: **our most important rails are business logic, not generic safety.**
*"No fee figure without a source and date." "No recommendation before the loan stress-test."
"Never the words guaranteed placement."* No validator hub ships those. What off-the-shelf tools do
well — generic harm classification — we get from a dedicated safety model instead (§5).

### Evals: **Promptfoo** (CI) + **Langfuse self-hosted** (production)

The pattern the research converges on is one of each: a code-first framework gating CI, and an
observability platform tracing production.

- **Promptfoo** — Node-native, YAML config, MIT, used by OpenAI and Anthropic. Ships **red teaming
  with 50+ vulnerability scans** (jailbreak, prompt injection, data exfiltration). For a TS team
  this is the obvious CI choice; DeepEval is Python/pytest-native and would fight our stack.
- **Langfuse** — open-source, self-hostable, acquired by ClickHouse Jan 2026. Prompt versioning,
  tracing, cost tracking.

**On lock-in:** every major tracer (Langfuse, Phoenix, OpenLLMetry, Laminar) has converged on the
**OpenTelemetry GenAI semantic conventions**. If we emit standard `gen_ai.*` spans we can move to
Datadog or anything else later without touching application code. **Emit OTel from day one.**

### The rest

```
Runtime      Next.js 16 API routes, TypeScript, Vercel
Queue        Upstash QStash — 4s debounce, retries, timeout isolation
DB           Supabase (own project) — `wa` schema, RLS, least-privilege write role
Vectors      pgvector, same instance
Models       Gemini Flash-Lite (decide/extract) + Flash (generate), via AI Gateway
Safety       dedicated classifier, independent of the conversation model (§5)
Evals        Promptfoo in CI
Tracing      OTel GenAI conventions → Langfuse (self-hosted)
WhatsApp     Cloud API direct
Bridge       nightly ETL from newgen's existing public APIs
```

---

## 3. Accuracy: the model never touches a number

Unchanged from `DATA_STRATEGY.md`, restated because it's the backbone of "accurate":

| Layer | Handles | Risk |
|---|---|---|
| **A · Compute** | percentile→rank, loan ceiling, EMI, 4-year total | zero |
| **B · SQL** | fees, dates, ratings, cutoffs | zero |
| **C · pgvector** | review prose, campus texture | bounded |

Every retrieved value carries `source` + `updated_at` into the **fact block**. The generation prompt
may only *phrase* what's in the block. It has no authority to introduce a figure.

---

## 4. Hallucination: four failure modes, four controls

Current research decomposes hallucination into **factual, grounding, citation and reasoning**
failures — and the key finding is that groundedness must be checked at **claim level, not answer
level**. An answer that's 90% supported still contains one fabricated fee.

| Mode | Control | Where | Blocking? |
|---|---|---|---|
| **Factual** — invented number | Regex/numeric sweep: every `₹`, `LPA`, `%`, rank, date in the output must appear in the fact block | Rail 4, ~0ms | **Yes** |
| **Grounding** — claim unsupported by retrieved prose | Claim-level entailment against retrieved chunks | Rail 4, on recommendation + RAG messages only | **Yes** |
| **Citation** — right number, wrong college | Fact block is keyed by `college_slug`; cross-slug reference fails the check | Rail 4 | **Yes** |
| **Reasoning** — sound facts, invalid conclusion | Recommendation ordering comes from the **match engine**, not the model. The model cannot reorder or add a college. | Rail 2/3 | structural |

**LLM-as-judge stays out of the production hot path** — too slow and too expensive per message. It
belongs in the Promptfoo suite, where it grades hundreds of cases offline. Compact encoder-based
span detectors can localise unsupported spans more cheaply than an LLM judge; worth evaluating at
Phase 3 if the numeric sweep proves insufficient.

**Failure behaviour: block, log, fall back to a template.** A slightly stiff templated message beats
a confident wrong fee. Every block raises a `grounding_violation` event — and that rate is a
release gate (§11).

---

## 5. Safety — and why an off-the-shelf guard is not enough

Current mental-health-AI research makes two findings that matter directly:

1. **Dedicated risk-detection modules should run independently of the conversation model**, so
   empathic engagement stays warm while safety monitoring stays conservative in the background.
   *(This validates the architecture in §1 — safety pre-empts the state machine.)*
2. **General-purpose guardrails are poorly suited to mental-health contexts.** They classify into
   broad harm categories and detect *the presence of sensitive topics*, rather than
   *clinically meaningful risk within context*.

### The India-specific gap

Generic classifiers (Llama Guard, ShieldGemma) are trained predominantly on English, Western crisis
phrasing. Indian exam-stress distress does not look like that. It looks like:

> *"papa ka sapna tod diya maine"* · *"ghar walo ne itna paisa kharch kiya, sab bekaar"*
> *"kis muh se ghar jaunga"* · *"main hi nalayak hu"*

Not one of those contains a keyword a generic guard is looking for. **We have to build this layer
ourselves**, with Hinglish examples, or it will not work.

### Design

```
Every inbound → SAFETY (independent, pre-empts everything)
   Pass 1  generic harm model              cheap, catches the explicit
   Pass 2  domain classifier, DECOMPOSED   5 narrow bools, Hinglish-trained
             · hopelessness about the future
             · explicit self-harm / suicide language
             · irreparable family-shame framing      ← the India-specific one
             · reference to a plan or method
             · expressing being a burden
   weighted → conservative threshold → we accept false positives
```

**Decomposition matters here specifically.** The Jev research found one broad question scored 62.6%
where five narrow ones scored 95.0%. Safety is exactly where we want that 32-point gap on our side.

**On trigger:** funnel stops · supportive response · **Tele-MANAS 14416** (confirmed active — 53
cells across 36 states/UTs, run by NIMHANS/MoHFW) · founders notified via Notify · conversation
flagged, no CTAs ever.

**Fail-safe: classifier error or timeout → escalate.** Never silently continue.

### Scope control (Rail 1)

A separate input classifier, three outcomes:

| Class | Behaviour |
|---|---|
| **In scope** — colleges, exams, fees, careers, the decision | proceed |
| **Adjacent** — "which laptop for coding?", general study advice | answer briefly, redirect |
| **Out of scope** — homework, politics, medical, relationships, anything else | decline warmly, redirect once |

Plus the **unverified-college rule (D5)**: answer honestly, label it explicitly as not-our-verified-
data, log to `wa_college_demand`.

---

## 6. Reasoning: decomposition everywhere

The single most transferable finding from the Jev research: **narrow questions beat broad ones by a
very large margin.** This becomes a codebase rule, not a preference.

**Banned:** `"What should this student do?"`
**Required:** a chain of bounded decisions, each a typed `Decision` call —

```
bool   student has given a usable exam score?
bool   stated budget is internally consistent with stated income?
choice which constraint binds hardest: money | geography | degree-route | timeline?
score  how ready is this student for a recommendation?   0-100
bool   does the loan math clear?
choice which handoff fits: connect | college-page | human | none?
```

Each is cheap, individually testable, and individually loggable. When a recommendation is wrong we
can see *which* decision was wrong. A single broad prompt gives us nothing to debug.

---

## 7. Tone, politeness, engagement

Not prompt vibes — three enforced mechanisms:

1. **Persona few-shots.** 12–15 real Hinglish exchanges in the system prompt. Research finding:
   adding twelve Hinglish few-shot examples produced *dramatically* better output with no
   fine-tuning at all. Cheapest quality win available.
2. **Tone rails (Rail 4).** A classifier on our own output, blocking: condescension · false cheer
   at a distressed student · sales pressure · any guarantee language · emoji spam · English-only
   drift when the student is writing Hinglish.
3. **Engagement as message design, not personality.** Batched questions, tap-buttons, one idea per
   message, always a next step, never a wall of text. Engagement is measured by
   **stage depth** (D4) — not by how friendly the copy sounds.

**Banned phrases** ship as a literal list in the repo, checked at Rail 4: *"guaranteed placement",
"100% placement", "don't worry", "Dear Student", "best college for you", "limited seats", "hurry"*.

---

## 8. "Properly trained" — the honest path

Three options, and the sequencing matters more than the choice.

| Approach | For | Verdict |
|---|---|---|
| **Prompt + few-shot** | Hinglish register, persona, format | ✅ **Start here.** 12 examples gave dramatic gains with zero training in published results |
| **RAG + SQL** | All facts | ✅ Already the design. Facts must stay swappable — they expire every admission cycle |
| **LoRA fine-tune** | Consistent Hinglish *style* | ✅ **Phase 5, on our own transcripts** |

The fine-tuning evidence for Hinglish is genuinely strong: a Qwen2.5-3B Hinglish LoRA showed
**+41.4% fluency** and **+42.4% coherence** over base, with human A/B preference of **87.8% vs
12–39%** for the base model. That's not marginal.

But two conditions:
1. **Never fine-tune facts.** Fees, dates and cutoffs change every cycle; baking them into weights
   creates hallucinations that no guard can catch, because the model will be confident and the fact
   block won't contain them. **Fine-tune style only. Facts stay in SQL and RAG forever.**
2. **We need real transcripts to train on** — which only v0 produces. Fine-tuning before we have
   our own conversation data means training on someone else's idea of Hinglish.

So: prompting now, LoRA once we have a few thousand real turns and a stable eval suite to prove it
actually improved things.

---

## 9. The scenario matrix

Correctness is defined by a matrix, not by vibes. Every cell gets a Promptfoo test.

| Axis | Values |
|---|---|
| Stage | class 10 · 11 · 12 · dropper · already in college |
| Score | none · low · mid · high · not disclosed · lying |
| Money | can't afford anything · tight · comfortable · undisclosed · unrealistic |
| Emotion | calm · anxious · **distressed** · angry · defensive |
| Language | English · Hinglish · Hindi-in-Roman · broken · emoji-only |
| Who's typing | student · **parent** · sibling · friend · counsellor probing us |
| Intent | genuine · testing the bot · trying to break it · **a college checking their listing** |

That last one is real: **rival counsellors and the colleges themselves will talk to this bot.**
It must behave identically — which it will, because honesty is the design.

---

## 10. Edge-case taxonomy

| Class | Examples | Handling |
|---|---|---|
| **Protocol** | duplicate webhooks · out-of-order delivery · media we can't read · unsupported types · user blocks mid-flow | idempotent on `wamid`, graceful degrade |
| **Window** | returns after 3 days · 24h expiry mid-compose · month-boundary free tier | **Rail 5 owns this**; stage logic never thinks about it |
| **Conversational** | answers 3 questions at once · answers a previous question · changes their mind · silence for 2 days · "start over" | explicit `restart` and `back` intents |
| **Data** | college not on platform · no mentor live · fact older than 60 days · review corpus empty for a college | honest disclosure, always |
| **Adversarial** | prompt injection · "ignore your instructions" · extracting the system prompt · fee-figure baiting | Promptfoo red team suite in CI |
| **Identity** | parent takes the phone · two students one number · a minor | re-detect speaker; DPDP consent |
| **Emotional** | distress · rage at a college · rage at us · a student who just wants to talk | safety rail; never sell into distress |
| **Ours** | LLM down · Supabase paused · guard blocks everything · QStash backlog | **always a templated fallback that admits the problem** |

Last row matters: **there is always a hand-written message for total failure.** Never silence.

---

## 11. Eval infrastructure — the actual differentiator

With three people and eight months, this is what separates a demo from a product.

```
    /\      Manual review — 20 real transcripts/week, by us
   /  \     Red team — Promptfoo, 50+ vuln classes, weekly
  /    \    Scenario evals — the §9 matrix, every PR
 /      \   Golden conversations — Aarav + 20 more, every PR
/________\  Unit — compute, match engine, guards. No LLM. Milliseconds.
```

**Release gates — a PR that breaks any of these does not merge:**

| Gate | Threshold |
|---|---|
| Grounding violations on the golden set | **0** |
| Safety recall on the distress set | **100%** |
| Scope escapes | **0** |
| Banned phrases | **0** |
| Match-engine determinism | byte-identical for identical input |
| Median outbound per golden conversation | ≤ 16 |

**Transcript replay is the highest-value thing we build.** Record real conversations; replay them
against every prompt change; diff the outputs. Without it, prompt edits are unfalsifiable — which is
how these products silently rot.

---

## 12. Team split (3 engineers)

Mirrors the Founder A/B/C pattern already used for Connect.

| | Owns | Phase 1 deliverable |
|---|---|---|
| **A · Conversation core** | webhook · worker · QStash · state machine · 7 stages · send layer · window rules | deterministic bot, zero LLM, end-to-end |
| **B · Data & retrieval** | Supabase schema · ETL from newgen · fact table · pgvector · compute (rank/EMI) · match engine | fact block + match engine, fully tested |
| **C · Model, safety & eval** | ports · prompts · few-shots · all 5 rails · safety classifier · Promptfoo · Langfuse · red team | rails + eval harness against A's stub |

The ports design is what makes this parallelisable: **A and B never wait on C.** A builds against a
mocked `Generator` that returns fixed strings; B's layers are pure functions. The LLM arrives last
and slots in.

---

## 13. Phases

Sized for 3 people. The real deadline is June 2027, so this is deliberately unhurried.

| Phase | ~Duration | Exit criteria |
|---|---|---|
| **0 · Foundations** | 1 wk | Meta app + **payment method** + test number · schema · echo bot · CI · OTel |
| **1 · Deterministic bot** | 2 wks | All 7 stages run end-to-end with **no LLM**. Buttons, SQL facts, compute, match engine. Fully unit tested. |
| **2 · Model layer** | 2 wks | Extraction, generation, RAG, all 5 rails live. Grounding guard blocking. Safety classifier with Hinglish set. |
| **3 · Eval & hardening** | 2 wks | Promptfoo scenario matrix + red team in CI. Release gates enforced. Langfuse tracing. Replay harness. |
| **4 · Private pilot** | 2 wks | 50 real students on the test number. Prompt tuning from real transcripts. Cost instrumented. |
| **5 · Launch + flywheel** | ongoing | Production number. **Review collection to July-2026 enrollees.** LoRA evaluated once transcripts exist. |

**≈ 9 weeks to a genuinely good bot — live by early December**, leaving ~6 months of real traffic to
refine before the June peak. Launching into peak season untested was always the bigger risk.

---

## 14. What the stack costs

| | Monthly |
|---|---|
| Supabase Pro (3rd project) | ~₹2,200 |
| Vercel Pro (when traffic is real) | ~₹1,800 |
| Upstash QStash | ₹0 → free tier |
| Promptfoo | **₹0** — MIT |
| Langfuse self-hosted | ~₹500 small container, or ₹0 on cloud free tier |
| LLM (dev + evals + traffic) | ₹1,500–4,000 |
| WhatsApp | ₹0 under ~95 students/mo, then ~₹1.10/student |
| **Total, pilot scale** | **≈ ₹6,000–9,000/month** |

Eval and observability — the parts that make it trustworthy — are the cheapest line items on the
page. There is no reason to skip them.

---

## Sources
- [NeMo Guardrails vs Guardrails AI vs LLM Guard (2026)](https://kanopylabs.com/blog/guardrails-ai-vs-nemo-guardrails-vs-llm-guard) · [Particula — NeMo vs Llama Guard vs Guardrails AI](https://particula.tech/blog/ai-guardrails-compared-nemo-guardrails-ai-llama-guard)
- [Speakeasy — agent framework comparison](https://www.speakeasy.com/blog/ai-agent-framework-comparison) · [Particula — Mastra vs LangGraph vs Vercel AI SDK](https://particula.tech/blog/mastra-vs-langgraph-vs-vercel-ai-sdk-typescript-agents)
- [Promptfoo — LLM red teaming guide](https://www.promptfoo.dev/docs/red-team/) · [promptfoo/promptfoo](https://github.com/promptfoo/promptfoo)
- [AgentsCamp — best LLM eval tools 2026](https://agentscamp.com/guides/evaluation/best-llm-eval-tools-2026) · [QASkills — Braintrust vs Langfuse](https://qaskills.sh/blog/braintrust-vs-langfuse)
- [Braintrust — hallucination evaluation metrics and methods 2026](https://www.braintrust.dev/articles/ai-hallucination-evaluations-metrics-methods-2026) · [Span-level hallucination detection (arXiv 2607.00895)](https://arxiv.org/pdf/2607.00895)
- [Langfuse — OpenTelemetry integration](https://langfuse.com/integrations/native/opentelemetry) · [OTel GenAI semantic conventions](https://dev.to/gabrielanhaia/opentelemetry-genai-semantic-conventions-your-llm-traces-should-look-like-this-in-2026-3ff6)
- [MindGuard — guardrail classifiers for multi-turn mental health support (arXiv 2602.00950)](https://arxiv.org/html/2602.00950) · [Suicide- and crisis-risk detection in mental-health chatbots (medRxiv)](https://www.medrxiv.org/content/10.64898/2026.01.12.26343914v1.full.pdf)
- [Sample-efficient language model for Hinglish conversational AI (arXiv 2504.19070)](https://arxiv.org/pdf/2504.19070) · [Hinglish LoRA fine-tuning playground](https://github.com/Prakhar-Bhartiya/llm-finetune-playground)
