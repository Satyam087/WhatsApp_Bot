# Campus Critique WhatsApp Bot — Product Thesis
### What it's for, who it serves, what it does, and how we know it worked

Draft 2026-09-24 · **decisions resolved — see `DECISIONS.md`** · supersedes the product assumptions in `PLAN_V0.md`

---

## Part 1 — The market, from the new-age college perspective

Before deciding what the bot does, here is what the numbers actually say.

### 1.1 The funnel that creates our user

| Fact | Number |
|---|---|
| JEE Main 2026 registrations | **~14.5–16 lakh** (record high) |
| Total JoSAA seats — IITs + NITs + IIITs + GFTIs | **67,323** |
| **Students who will NOT get a JoSAA seat** | **~95%** — roughly **13–15 lakh a year** |
| Droppers who meaningfully improve their rank | **~20–30%** |

That ~13 lakh figure is the addressable market. Every one of them, in a six-week window between
results and admission deadlines, has to make a ₹5–30 lakh decision with no reliable advisor.

### 1.2 What the 8 new-age colleges on our platform actually are

| College | Est. | Fees (4 yr) | Batch | Outcome history | Degree route |
|---|---|---|---|---|---|
| Newton (ADYPU) | 2024 | ₹17.5–26.9L | ~100 | Early signals | ADYPU |
| Newton (Rishihood) | 2023 | ₹17.5–26.9L | ~120 | Internships only | Rishihood |
| Scaler SST | 2023 | ₹17–24.5L | ~60–80 | Audited internships (96.3%) | **Separate HEI — verify** |
| Vedam | 2025 | ₹18L | ~60–80 | **None** | ADYPU |
| Intellipaat IST | 2025 | ₹16L | 63 | **None** (24 *unpaid* internships) | S-VYASA (deemed) |
| NIAT | 2023 | ₹8–18L | ~80 | 8.5 LPA avg / 24 LPA high | ADYPU |
| Veloces | 2024 | ₹10–16L | — | **None** | ADYPU |
| Alta | 2026 | ₹9.8–15L | — | **None** | ADYPU |

**Read that table again.** Five of eight have effectively no placement history, and several ask
₹16–27 lakh. The category's central question is not "which one is best" — it is
**"should I bet a fifth of my parents' net worth on an institution with no track record?"**

That is the question our bot exists to help answer honestly. Not to dodge.

### 1.3 The three things nobody tells these students

**(a) The degree is not what they think it is.**
Neither Scaler nor Newton is itself a UGC-recognised university. Degrees come via partner
institutions. For a pure software career this may not matter at all. For UPSC, PSUs, GATE,
some MS admissions, or a government job — it can matter enormously. Students do not know
to ask, and marketing pages are engineered so they don't.

**(b) The loan math almost never works, and the parent carries the risk.**
- Banks generally cap education loans at **~4× parental annual income**. A family earning ₹5L/yr
  cannot borrow ₹20L, whatever the brochure implies.
- Loans above ₹7.5L usually need **collateral — typically the family home**.
- On default (3 missed EMIs), the **co-applicant parent's CIBIL score drops 100–150 points and
  stays damaged for 7 years.**

Nobody in this market runs this arithmetic with the student. It takes ninety seconds, and it is the
single most consequential thing anyone could tell them.

**(c) Everyone advising them is paid per admission.**
Consultancies and portals earn per successful referral. Coaching institutes need drop-year
enrolments. College counsellors are salespeople. YouTube reviewers take sponsorships.
**A 17-year-old making the largest financial decision of their family's life is surrounded
exclusively by people with a financial stake in the answer.**

### 1.4 Where that leaves Campus Critique

Our one irreplaceable asset is being **the only party in this market that is not paid per admission**.

Everything below is designed to keep that structurally true rather than merely claimed.

---

## Part 2 — The student's real problem (we had it wrong)

`PLAN_V0.md` assumed the problem was *"which college should I pick."* Having looked at the
category properly, that's the last 10% of the problem. The actual state of a student who opens
this chat in June:

| What they say | What's really happening |
|---|---|
| "78 percentile aaya, kuch nahi ho sakta ab" | **Emotional collapse.** Cannot absorb information in this state. |
| "Sab alag alag bol rahe hain" | **Information chaos.** Coaching says drop, parents say govt college, YouTube says Scaler, uncle says "private college mat lena." |
| "Papa puch rahe hain 20 lakh worth it hai?" | **A financial decision they are not equipped to make** — and they're the one being asked. |
| "Degree valid hai na?" | **Fear of being scammed**, with no way to verify. |
| "IIT nahi to kuch nahi" | **Their option space is tiny and wrong.** They can't see the map. |

So the job is not recommendation. The job is:

> **Take a panicking, badly-informed 17-year-old and turn them into a calm one who understands
> their real options, the real trade-offs, and can defend a decision to their parents.**

A college recommendation is sometimes an output. It is not the point.

---

## Part 3 — Objective (the uncomfortable part)

There are three possible objectives and they genuinely conflict. We should pick deliberately.

| | **A — Lead engine** | **B — Public service** | **C — Decision infrastructure** |
|---|---|---|---|
| Bot's goal | Qualify → sell to the 8 colleges | Help everyone, monetise nothing | Make the decision *good*; monetise trust |
| Revenue | ₹20–60k per admission | None | Connect, partnerships, data, brand |
| Problem | **This is exactly the industry we exist to replace.** Students smell it. The moat dies. | Doesn't survive | Slower, needs discipline |

**Recommendation: C, and it needs one structural commitment to be real —**

> ### The bot's success metric must never be admissions.
> The moment a rupee flows per admission, recommendations drift, and we become the thing we were
> built to displace. Our defensibility *is* our disinterest.

### The test of whether we actually mean it

**The bot must be capable of saying: "Based on your family's finances, you should not spend
₹20 lakh on any of these. Here's what I'd do instead."**

If that sentence is impossible for our bot to produce, we have built a sales funnel with better copy.
If it is possible — and occasionally happens — then every *positive* recommendation becomes credible,
because the student knows the bot was willing to say no.

I'd go further: **% of conversations where the bot advised against spending is a KPI we want to be
non-zero.** Track it on the dashboard next to leads.

### So what do we actually get out of it?

1. **Trust at the moment of maximum vulnerability.** That student tells five friends and comes back
   for their sibling.
2. **Warm, honest, well-matched leads** — worth *more* to a partner college than volume, because
   they don't drop out in year one.
3. **Connect bookings** — the natural next step from "I don't have verified data on that" is
   "want to ask an actual student? ₹99."
4. **The review flywheel** (Part 6) — the bot becomes our lowest-friction review-collection channel.
5. **Demand data nobody else has** — real budgets, real anxieties, real questions, at scale.

---

## Part 4 — How the bot works: the 7-stage model

Not a form. Not a chatbot flowchart. A conversation with a structure a good senior would follow.

```
 1. STABILISE     → deal with the emotional state before any information
 2. LOCATE        → where does this student actually stand (scores, money, family, intent)
 3. WIDEN         → show the real option map; most arrive thinking they have 2 options
 4. STRESS-TEST   → the loan math, the no-placement scenario, the degree question   ★ differentiator
 5. NARROW        → now recommend, with one honest caution per college
 6. EQUIP         → the "Ask This" pack: exact questions to ask, what a good answer sounds like
 7. HAND OFF      → verified senior / college page / human counsellor / nothing, if nothing is right
```

**Stage 4 and Stage 6 are the whole product.** Any competitor can do 1, 2, 5, 7.

### Stage 1 — Stabilise
You cannot give information to someone in panic. Validate specifically (never "don't worry"),
normalise with a real number, reframe, then one micro-step.

> "78 percentile after a full year of prep genuinely stings. Not going to pretend otherwise.
> Here's a number that might help though — of the ~14.5 lakh who wrote JEE Main this year,
> about 13 lakh won't get a JoSAA seat either. You're not the exception. You're the norm,
> and the norm has options. Want to see them?"

**Hard rule:** distress / self-harm language → funnel stops immediately, supportive reply,
**Tele-MANAS 14416**, human flag. No recommendations, no CTAs, ever.

### Stage 3 — Widen (the most under-rated stage)
Most students arrive believing their choice is "drop, or a bad private college." The real map:

```
 JoSAA rounds 5-6 / spot rounds   ·   State quota & home-state colleges
 Tier-1 private (VIT/BITS/SRM/Thapar via their own exams)
 New-age colleges (our 8)         ·   Drop year (with honest 20-30% odds)
 BSc CS / BCA + self-taught route ·   Tier-2 private + GATE plan
```

We only have deep data on one branch. **We should still show the whole map.** Honesty about what
we don't cover is the cheapest trust we will ever buy.

### Stage 4 — Stress-test ★
Three tests, run conversationally, none of which anyone else in this market runs:

**The loan test**
> "Rough number, no need to be exact — what's your family's annual income?"
> → "₹6L. So banks will likely sanction up to ~₹24L, but anything over ₹7.5L needs collateral —
> usually your house. Newton at ₹26.9L is above what most banks will clear without your parents
> putting property up. Is that a conversation your family has actually had?"

**The downside test**
> "Say you graduate and don't get placed in six months. What happens? Because at ₹18L over 7 years,
> that's roughly ₹28–30k a month in EMI starting six months after you finish — and if it's missed,
> it hits your father's CIBIL, not just yours, for seven years."

**The degree test**
> "Quick one that matters more than people think — do you see yourself possibly doing GATE, a
> government job, or an MS abroad? Because these colleges award degrees through partner universities,
> and some of those routes are scrutinised differently. For pure software jobs it's a non-issue."

These are not upsell questions. They are the questions a senior who actually cares would ask, and
asking them is how we earn the right to recommend anything.

### Stage 6 — Equip (the brand-defining stage)
The student leaves with a personalised **"Ask This" pack** — even if they never come back:

> **Before you pay Vedam's deposit, ask admissions these 5, and text me what they say:**
> 1. "Which university is on my final degree certificate, and is it UGC-recognised?"
>    *Good answer: a specific named university you can verify. Bad answer: "it's fully valid, sir."*
> 2. "How many of your 2025 batch got paid internships, and what was the median stipend?"
>    *Bad answer: only the highest number.*
> 3. "Is the ₹18L all-in? Hostel, mess, exam, caution deposit?"
> 4. "What's the refund policy if I leave after semester 1?"
> 5. "Can I speak to a current student you haven't selected for me?"

A student who leaves with this has been genuinely helped whether or not they ever convert.
It also quietly makes every college on our platform behave better.

---

## Part 5 — Capabilities

### Tier 1 — v0 (must exist at launch)
| # | Capability | Why |
|---|---|---|
| 1 | **Emotional triage + safety path** | Users are stressed minors. Non-negotiable. |
| 2 | **Profile building** — stage, scores, money, geography, intent, risk appetite | Everything downstream needs it |
| 3 | **Option-map widening** | Cheapest trust we can buy |
| 4 | **Affordability stress-test** ★ | Nobody else does this |
| 5 | **Grounded matching over the 8 campuses + one honest caution each** | Our data moat |
| 6 | **"Ask This" pack** ★ | Brand-defining; helps even non-converters |
| 7 | **Handoff** — Connect / college page / human / *nothing* | "Nothing" must be a real option |
| 8 | **Honest refusal** — "I don't have verified data on that" | The credibility valve |

### Tier 2 — v1 (the compounding ones)
| # | Capability | Why it matters |
|---|---|---|
| 12 | **Review collection over WhatsApp** ★★ | **Moved to the front of v1.** Students who joined in July 2026 are 2–3 months in right now — the sweet spot. Vastly lower friction than the web wizard, and it doesn't depend on the counselling funnel. See `DATA_STRATEGY.md` §4. |
| 10 | **Deadline tracking** | Reuses the admission-alert cron we already run. "VSAT is in 6 days, you haven't applied." |
| 14 | **Side-by-side compare on demand** | Reuses `collegeCompareProfiles` |
| 11 | **Post-decision follow-up** | 3/6/12 months: "How's it actually going?" → feeds #12 and Part 6 |
| 13 | **Claim checker** | Student forwards what a counsellor told them; we check it against verified data |
| — | **JoSAA cutoff lookup** | Rank-band prediction with confidence bands. Timed for well before June 2027. |

### Tier 3 — v2 and later
| # | Capability | Why it matters |
|---|---|---|
| 9 | **Parent mode** ★★ | The parent signs the loan and carries the CIBIL risk, and nobody in this market talks to them. Deferred to v2 by decision, but it remains the single biggest untapped surface. |
| — | Cohort insights, scholarship & fee-negotiation guidance, regional languages, alumni-outcome tracking | |

---

## Part 6 — The flywheel (why this compounds)

```
   Confused student  ──▶  Bot helps honestly  ──▶  Student enrols somewhere
          ▲                                                  │
          │                                                  ▼
   Bot gets smarter  ◀──  Verified reviews  ◀──  Bot follows up at 3/6/12 months
   for next year's                                 "How is it actually going?"
   students                                        (WhatsApp, not a web form)
```

Two things make this real and not a slide:

1. **Review collection is currently our bottleneck.** Today it needs a verified college email plus a
   multi-step web wizard. A WhatsApp follow-up to a student we already helped — who already trusts
   us — is an order of magnitude lower friction. The bot is the best review-intake channel we could
   build, and it's the same bot.

2. **It makes our honesty auditable.** If we warned a student "Intellipaat's robotics lab is still
   under construction" and six months later that student confirms it, our caution data is *verified
   by outcome*. Prediction accuracy becomes a measurable, publishable trust asset. No competitor can
   fake it, because it requires having told students things a year earlier and being right.

---

## Part 7 — How we know we helped (the measurement question)

> **Decided (D4):** we are **not** asking students a confidence question or running a 6-month
> regret check-in. Everything below is **derived from the transcript** — zero extra messages,
> zero friction, zero cost. Full metric table and the trade-off this involves: `DECISIONS.md`.

### What we track
| Metric | Derived from | Healthy |
|---|---|---|
| **Stage depth** | Deepest of the 7 stages reached | Median ≥5 |
| **Stress-test completion** | Did Stage 4 actually run — loan + downside + degree? | ≥60% |
| **Option Set Expansion** | Options named at intake vs. present at close | ≥1 new in 70% |
| **Myth corrections** | False beliefs corrected | tracked |
| **Ask-This delivered** | Stage 6 pack sent | ≥50% of completed chats |
| **Handoff action** | College page click / Connect booking / pack saved | tracked |
| **Return rate** | Unprompted message within 30 days | ≥20% |
| **Unprompted referral** | New contact says a friend sent them | tracked |

### Guardrail metrics — watch these as closely as the growth ones
| Metric | Healthy range | Why |
|---|---|---|
| **`advised_against_spending_rate`** ★ | **>0%, expect 10–20%** | If this reads 0% for a month, we built a sales funnel. P0 bug. |
| **`dont_know_rate`** | >0% | Honest refusal is a feature, not a defect |
| **Ungrounded numeric claims** | **0** | One fake "₹45 LPA average" screenshot ends the brand |
| **Distress flags correctly routed** | **100%** | Non-negotiable |
| **Median outbound messages** | ≤16 | Cost discipline (`PLAN_V0.md` §3.2) |

**The trade-off we accepted:** these prove the conversation was *thorough*. They cannot prove the
student ended up *better off* — that needed the before/after question or the 6-month check-in.
We are using engagement quality as a proxy for help, knowingly.

## Part 8 — Hard rules (what the bot must never do)

1. Never claim or imply guaranteed placement or admission.
2. Never state a fee, package, date or stat that isn't in our sourced dataset — with a date on it.
3. Never push a college when the loan math doesn't work.
4. Never pretend to be a human. It is a Campus Critique bot; if asked, it says so.
5. Never continue the funnel after a distress signal.
6. Never collect more personal data than the current stage requires (DPDP — most users are minors).
7. Never tell a student what to do. Give them the trade-off and the questions. **The decision is theirs
   and their parents'.**
8. Always be willing to say "none of these are right for you."

---

## Part 9 — Decisions (resolved 2026-09-24)

Full record in `DECISIONS.md`.

| # | Question | Answer |
|---|---|---|
| 1 | Commit to Objective C — success metric never admissions? | ✅ **Yes, locked.** No per-admission revenue, ever. `advised_against_spending_rate` ships on the dashboard from day one. |
| 2 | May the bot recommend outside our 8 colleges? | ✅ **Yes.** JoSAA, state quota, tier-1 private, drop year, BSc+self-taught all in scope. |
| 3 | Parent mode — v1 or v2? | **v2.** Deadline tracking (#10) and compare-on-demand (#14) confirmed for **v1**. |
| 4 | Adopt Confidence Delta + 6-month regret as dashboarded metrics? | ❌ **No.** Transcript-derived measurement instead — see Part 7. |
| 5 | Persona / handoff | **Handoff = a Connect mentor from the college the student actually chose.** Unlisted college → "we're adding this soon" + log to `wa_college_demand`. Naming still open. |
| 6 | v0 scope | **All 7 stages.** Complete as much as possible in week 1. |
| 7 | JEE data — integrate, or is RAG enough? | **Neither alone.** Deterministic compute + SQL for numbers, RAG for prose. JoSAA table → v1. See `DATA_STRATEGY.md`. |
| 8 | Jev, the new decision model? | **Architect for it, don't depend on it yet.** `Decision` port now, shadow-eval later. Adopt decomposition immediately. |
| 9 | Standalone project, or inside Campus Critique? | **Fully standalone** — own repo, Vercel, Supabase, WABA. Nightly ETL bridge. |
| 10 | Cap long conversations to control cost? | **No cap.** A 12-month power user costs ~₹12. |
| 11 | Build timeline | **~9 weeks, 3 engineers.** Phase 1 = working bot with zero LLM. |

### The demand log is the sleeper feature
Every time a student names a college we don't cover, that's an unprompted, high-intent vote for
which college page to build next — ranked by real demand instead of guesswork. Decision 5 turns our
biggest coverage gap into a roadmap input no competitor has.

## Sources
- [Collegedunia — JEE Main 2026: 14.5 lakh students, 58,000 seats](https://collegedunia.com/articles/e-301-jee-main-2026-14-lakh-50-thousand-students-for-58000-seats)
- [Collegedunia — JEE Main 2026 seat matrix (67,323 seats)](https://collegedunia.com/exams/jee-main/seat-matrix)
- [ALLEN — JEE Main 2026 crosses 1.6 million applications](https://news.allen.in/jee-main-2026-hits-record-over-1-point-6-million-applications/)
- [Aakash — Drop year vs private college 2026](https://www.aakash.ac.in/blog/drop-year-vs-private-college-admission-2026-jee-aspirants-guide/)
- [CareerWithMohit — Scaler vs Newton: the negative points](https://www.careerwithmohit.online/blog/scaler-vs-newton-school-of-technology-negative-review)
- [CollegeSimplified — Scaler vs Newton vs Masai 2026](https://www.collegesimplified.in/post/scaler-vs-newton-vs-masai-2026-comparing-modern-tech-school-courses-and-streams)
- [Tata Capital — ₹20 lakh education loan eligibility](https://www.tatacapital.com/blog/loan-for-education/can-i-get-a-20-lakhs-education-loan/)
- [DebtZen — Education loan default: consequences for student and guarantor](https://www.debtzen.in/education-loan-default-india)
- [Wikipedia — Capitation fee (India)](https://en.wikipedia.org/wiki/Capitation_fee)
- Internal: `newgen/lib/data.js`, `docs/context/FEATURE_STATUS.md`, `docs/product/connect_prd_v2.md`, `lib/admissions/staticAdmissionAlerts.js`
