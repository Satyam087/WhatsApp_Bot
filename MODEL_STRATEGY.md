# Model strategy — what the brain is actually made of

Answering: *"for making this agent intelligent and fast and strong, do we depend on deterministic
core + RAG + tree index only, or can we use a decision model like **Jev** too — not just LLMs?"*

I misread this as "JEE" last round. This is the better question, and the answer is yes to being
polyglot — with a caveat about where our actual bottleneck is.

---

## 1. What Jev is

TypeSafe AI came out of stealth **15 September 2026** — nine days ago — and shipped **Jev**, a
"System One" model. Founder Diogo Almeida is ex-OpenAI, co-author of the RLHF/InstructGPT work.

It is **not a language model**. It reads text and returns **typed decisions**, never prose:

| Output type | What it is |
|---|---|
| **Noul** | binary yes/no, with a probability |
| **Score** | graded rubric (e.g. 0–100 urgency) |
| **Choice** | categorical pick from a predefined set |

Every answer carries a **calibrated** confidence — trained via "Reinforcement Learning for Calibrated
Decisions". Calibrated means 0.8 actually happens ~80% of the time, which is the interesting part:
standard LLMs are notoriously overconfident, so you can't threshold on their stated probabilities.
With a calibrated model you can.

- **Latency:** 70–500ms (median ~0.32s) vs ~2.7s for LLMs
- **Cost:** $0.042/M input tokens, **no output charge** — output is a typed value, not tokens.
  ~$0.0004/decision, roughly $0.22 per 1,000 documents vs $1.31–3.08 for generative models
- **0% structured output error rate** by construction — outputs are mathematically bounded to the
  schema, so a type error is impossible. (That's the real meaning of "never hallucinates" — it's a
  claim about *format*, not about being *right*.)
- **Access:** open signup since ~20 Sept, $5 free credit. **Already in Vercel's AI Gateway as
  `typesafe-ai/jev`** — and we're on Vercel, so trying it costs nothing.

## 2. What independent testing actually found

This is where it gets more sober, and it matters more than the marketing.

| Claim | Independent finding |
|---|---|
| "193.6x faster, 444.6x cheaper" | **Does not reproduce** against real baselines. TypeSafe's *own employee* measured **15.9%** improvement on a real pipeline. |
| Accuracy | **67.8%** on their benchmark — level with mid-price LLMs, **5–6 points behind frontier**. 8 days of independent testing: *"level with mid-price LLMs, behind the frontier."* |
| Benchmark integrity | Reported accuracy is *agreement with a model-derived reference*, not verified correctness. Three factual errors found in their own table. |
| Real speed | 2.0–3.6x faster than baselines — real, but not two orders of magnitude |

**Bad at, per its own documentation:** arithmetic, dates, counting (*"unreliable, and the error grows
with size"*), structured extraction, images, and anything needing an explanation.
Also: *"English is where accuracy is best."*

### ★ The one genuinely valuable finding

On a 2,000-email phishing test, Jev scored **62.6% asked as one question — and 95.0% when split into
five narrow questions** with fitted weights.

**That 32-point jump is a property of decomposition, not of Jev.** It's the most useful thing in this
entire research thread, it applies to whatever model we use, and it's free. We should adopt it now.

---

## 3. Does it fit our bot?

### Where it could plug in
Our design already has a classification layer — currently "Gemini Flash-Lite doing routing". Jev maps
onto it cleanly:

| Our job | Jev type |
|---|---|
| Distress / safety detection | Noul, calibrated ★ |
| Which of the 7 stages next | Choice |
| Lead quality | Score |
| "Is this student confused?" | Noul |
| `advised_against_spending` detection | Noul |

### Where it structurally cannot
- **All generation** — every outbound message. It emits zero text.
- **Slot extraction** — `"78 percentile aaya bhai"` → `{exam: jee_main, percentile: 78.4}` is
  extraction, not a bounded choice. Stays with the LLM.
- **Arithmetic** — rank, EMI, loan ceiling. Its own docs say it's unreliable here. Confirms Layer A
  must stay in code, which we'd already decided.
- **Anything needing an audit trail** — it returns a number with no reasoning.

### Three reasons it doesn't belong on the v0 critical path

**1. The money isn't there.** Per conversation we spend ~₹1.61 on WhatsApp messaging and ~₹1.20 on
LLM, of which routing is maybe **₹0.30**. Jev takes that to ~₹0.05. **Saving ≈ ₹0.25/conversation —
about 8% of unit cost.** At v0 volume that's ₹50–250/month. That is not a reason to take a dependency
on a nine-day-old hosted-only proprietary vendor.

**2. Its headline advantage is worth nothing on WhatsApp.** 0.32s vs 2.7s matters for real-time
loops. We are **deliberately debouncing inbound messages by 4 seconds to save money** (`PLAN_V0.md`
§3.2). We are not latency-bound. Jev's main selling point does not apply to our channel.

**3. Hinglish.** *"English is where accuracy is best."* Our entire user base writes Roman-script
Hinglish. That's an unvalidated risk on precisely the dimension we need most — and nobody's
published a Hinglish eval.

---

## 4. Recommendation

> **Architect for it. Don't depend on it yet. Steal the decomposition technique immediately.**

**(a) Build the `Decision` port in v0 — regardless of vendor.**
Define one interface with three methods mirroring Jev's types:

```
choice(question, options, context)  → { value, confidence }
score (question, rubric,  context)  → { value, confidence }
bool  (question,          context)  → { value, confidence }
```

v0 implements it with Gemini Flash-Lite under constrained/structured output. Jev later becomes a
config change, not a refactor. Cost of building it this way: **zero** — and it forces bounded answer
spaces everywhere, the same discipline that makes our grounding guard work.

**(b) Adopt decomposition now.** Never ask one broad classification question. Split it into narrow
ones and combine. 62.6% → 95%. Works with any model. This is the actual intelligence upgrade.

**(c) Shadow-eval Jev in v1, on our data.** We're on Vercel, it's in the AI Gateway, there's $5 free
credit. Once we have ~500 real Hinglish transcripts, run both in parallel and decide on evidence
rather than a launch blog post. Not before we have data — there'd be nothing to measure against.

**(d) If it earns a place, safety is where it goes — but constrained.**
Calibration is a real, useful property for a distress classifier: you want a conservative threshold
you can actually trust (escalate above 0.15, not 0.5), and LLM confidence scores don't support that.
But Jev gives no reasoning, and safety is where we can least afford an unauditable call. So if we
adopt it there:

> **Jev runs as a second opinion that can only escalate, never de-escalate.** The LLM classifier stays
> primary and auditable. Jev can raise a case to the safety path; it can never clear one.

That gets the benefit of calibration with none of the explainability risk.

---

## 5. The bigger answer: the model layer is not our bottleneck

The full portfolio, with the right tool for each shape of problem:

| Layer | Tech | Why not an LLM |
|---|---|---|
| **Arithmetic** — rank, EMI, loan ceiling, 4-yr total | **Code** | Exact, free, auditable. Every model is unreliable here |
| **Facts** — fees, dates, cutoffs, ratings | **SQL** | Exact. Ordered comparison is a database's job |
| **Prose retrieval** — review text, campus texture | **pgvector, flat + metadata filter** | Genuinely the right tool |
| **Decisions** — routing, safety, scoring | **`Decision` port** → LLM now, Jev candidate | Swappable by design |
| **Generation** — every outbound message | **LLM (Gemini Flash)** | Only thing that can write Hinglish empathy |

**On tree index:** for 8 colleges and a few hundred reviews, a hierarchical tree index is
over-engineering. Flat pgvector with a metadata filter on `college_slug` + `section` will beat it at
our corpus size and is far simpler to debug. TreeIndex-style hierarchical summarisation earns its
place at 100k+ documents or deep document hierarchies. **Revisit when the review corpus passes
~10,000.** The flywheel is what gets us there, not a retrieval upgrade.

### The strategic point

A frontier model on bad data gives confident garbage. A mid-tier model on **excellent grounded data,
with a well-decomposed state machine**, gives genuinely good counselling.

What will make this bot strong is not a newer model. It is:
1. **Coverage and quality of verified review data** — the flywheel (`DATA_STRATEGY.md` §4)
2. **The structured fact table** with sources and dates
3. **Decomposition discipline** in the state machine

**Our moat is the data, not the inference.** Every hour spent picking models is an hour not spent
collecting reviews from students who enrolled in July and are three months in right now.

Jev is a real and well-made thing, and the `Decision` port means we can take it the moment it earns
its place. It just isn't the thing standing between us and a good bot.

---

## Sources
- [TechTarget — Jev decision model touted as quicker, cheaper LLM alternative](https://www.techtarget.com/it-infrastructure/news/366650696/Jev-decision-model-touted-as-quicker-cheaper-LLM-alternative)
- [Tom's Hardware — TypeSafe AI's Jev offers an alternative to LLMs](https://www.tomshardware.com/tech-industry/artificial-intelligence/typesafe-ais-jev-offers-an-alternative-to-llms-that-claims-to-be-193x-faster-and-445x-cheaper-system-one-type-model-is-bespoke-for-probabilistic-decision-making)
- [DataCamp — Jev: TypeSafe's System One model](https://www.datacamp.com/blog/system-one-models-jev)
- [DEV — Jev after eight days of independent tests: level with mid-price LLMs, behind the frontier](https://dev.to/gde/jev-after-eight-days-of-independent-tests-level-with-mid-price-llms-behind-the-frontier-1kln)
- [The Daily Brief — Jev scores 62.6% asked once, 95% split five ways](https://www.beri.net/article/typesafe-jev-typed-decision-model-calibration-decomposition-shadow-eval)
- [North Denver Tribune — TypeSafe claims 193.6x faster; its own employee measured 15.9%](https://northdenvertribune.com/news-2/jev-typesafe-benchmark-checked-explainer-wave/)
- [ejs-5/jev-benchmark — independent benchmark, 868 real decisions, mechanical labels](https://github.com/ejs-5/jev-benchmark)
- [APIMaster — Jev vs LLMs: where a decision model beats prompting a cheap model](https://apimaster.ai/blog/jev-vs-llm)
- [Vercel changelog — TypeSafe AI's Jev now available on AI Gateway](https://vercel.com/changelog/typesafe-ai-jev-now-available-on-ai-gateway)
