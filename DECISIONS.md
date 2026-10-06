# Locked decisions

Authoritative record. Anything in `PRODUCT_THESIS.md` or `PLAN_V0.md` that contradicts this is stale.

| # | Decision | Status | Date |
|---|---|---|---|
| D1 | **Objective C — decision infrastructure.** The bot's success metric is never admissions. No per-admission revenue, ever. | **Locked** | 2026-09-24 |
| D2 | **The bot may recommend outside our 8 colleges** — JoSAA rounds, state quota, tier-1 private, drop year, BSc+self-taught. | **Locked** | 2026-09-24 |
| D3 | **Parent mode → v2.** Deadline tracking (#10) and compare-on-demand (#14) confirmed for **v1**. | **Locked** | 2026-09-24 |
| D4 | **No Decision-Confidence-Delta / 6-month-regret metrics.** Measurement is transcript-derived instead — zero user burden. | **Locked** | 2026-09-24 |
| D5 | **Handoff = a Connect mentor from the college the student actually chose.** If that college isn't on the platform → "we're adding this college soon" + log the demand signal. | **Locked** | 2026-09-24 |
| D6 | **v0 ships all 7 stages.** Complete as much as we can in week 1. | **Locked** | 2026-09-24 |
| D7 | **JEE/JoSAA data: deterministic compute + SQL, not RAG.** See `DATA_STRATEGY.md`. JoSAA cutoff table lands in v1, not v0. | **Locked** | 2026-09-24 |
| D8 | **Jev (TypeSafe) — architect for it, don't depend on it yet.** Build a `Decision` port backed by an LLM; shadow-eval Jev later on our own Hinglish data. Adopt **decomposition** immediately. See `MODEL_STRATEGY.md`. | **Locked** | 2026-09-24 |
| D9 | **Built as a fully standalone project** — own repo, own Vercel project, own Supabase project, own WABA. Bridged to Campus Critique by a nightly ETL over existing public APIs. | **Locked** | 2026-09-24 |
| D10 | **No message cap, ever.** A 12-month power user costs ~₹12. Capping engaged students optimises a rounding error and caps exactly the students who convert, review and refer. | **Locked** | 2026-09-24 |
| D11 | **Stack: Vercel AI SDK (as a model library, not an agent framework) + own 5-rail guardrails in TS + Promptfoo in CI + Langfuse self-hosted on OTel.** See `ENGINEERING_ARCHITECTURE.md`. | **Locked** | 2026-09-24 |
| D12 | **~9-week phased build, 3 engineers — not 1 week.** Real deadline is the June 2027 peak. Phase 1 is a fully working bot with **zero LLM**. | **Locked** | 2026-09-24 |

---

## D1 — what "never admissions" means operationally

1. No per-admission or per-enrolment commercial agreement with any college on the platform. If one is
   offered, it is declined, and we say publicly that we declined it.
2. No internal target, OKR or dashboard tile counting admissions driven.
3. **`advised_against_spending_rate` ships on the dashboard from day one**, next to lead count. If it
   ever reads 0% for a full month, that is a P0 product bug, not a success.
4. The match engine has no field for commercial relationship. It cannot be weighted by one because
   the data doesn't exist in the table.

## D4 — what we measure instead, and what we gave up

Since we're not asking the student anything, **everything below is derived from the transcript**.
Zero extra messages, zero cost, zero friction.

| Metric | Derived from | Healthy |
|---|---|---|
| **Stage depth** | Deepest stage reached (1–7) | Median ≥5 |
| **Stress-test completion** | Did Stage 4 actually run — loan + downside + degree? | ≥60% |
| **Option Set Expansion** | Options the student named at intake vs. options present at close | ≥1 new in 70% |
| **Myth corrections** | Count of false beliefs the bot corrected | tracked, rising is fine |
| **Ask-This delivered** | Stage 6 pack sent | ≥50% of completed chats |
| **Handoff action** | Clicked a college page / booked Connect / saved the pack | tracked |
| **Return rate** | Student messages again unprompted within 30 days | ≥20% |
| **Unprompted referral** | A new contact says a friend sent them | tracked |
| **`advised_against_spending_rate`** ★ | Bot recommended not spending | **>0%, expect 10–20%** |
| **`dont_know_rate`** | Bot declined to answer for lack of verified data | >0% |
| **Ungrounded numeric claims** | Post-generation guard | **0** |
| **Distress routing** | Flagged → safety path | **100%** |

**The honest trade-off:** these prove the *conversation was thorough*. They cannot prove the
*student ended up better off* — that needed the before/after question or the 6-month check-in.
We're choosing engagement quality as a proxy for help. If you ever want the real signal back, the
cheapest version is one tap at the close of the chat (~₹0.115) — the offer stands, no need to decide now.

## D5 — handoff rules

```
Student converges on a college
        │
        ├── College IS on Campus Critique
        │      → college page deep link
        │      → "Want the unfiltered version? Talk to an actual <College> student.
        │         ₹99, 30 mins, they're not paid to sell you anything."
        │      → /connect deep-linked, filtered to that college
        │      → if no mentor is live for that college yet:
        │         "No <College> senior is available right now — I'll message you when one is."
        │         (queue it; we already have the mentor-eligibility system)
        │
        └── College is NOT on Campus Critique  (enabled by D2)
               → answer honestly from general knowledge, flagged as unverified
               → "I don't have verified student data on <College> yet — we're adding it soon.
                  I'll message you the day it's live."
               → log to `wa_college_demand`  ★
```

**The demand log is the point.** Every "not listed" is a student telling us, unprompted and with
real intent, which college page to build next — ranked by actual demand rather than guesswork.
That's a free product-roadmap input no competitor has, and it turns our biggest gap into an asset.

## D8 — the short version

Jev is real, well-made, and genuinely novel: typed decisions with *calibrated* probabilities, ~0.32s,
no output-token charge. But it's nine days old, its headline benchmarks don't reproduce independently
(its own employee measured 15.9% where marketing says 193.6x), it's level with mid-price LLMs rather
than ahead of frontier, and it's weakest on non-English — which is our entire user base.

It would save ~₹0.25/conversation, about **8% of unit cost**, and its speed advantage is worth nothing
on a channel where we deliberately debounce by 4 seconds.

So: build the swappable `Decision` port now (free), take the decomposition insight now (62.6% → 95%,
works with any model), and shadow-eval Jev once we have ~500 real Hinglish transcripts to judge it on.

## D9 — the bridge, in one picture

Full separation costs less than expected because most of the bridge already exists:

| Bot needs | Source | Effort |
|---|---|---|
| Approved reviews (RAG) | `/api/reviews/approved` — already public | none |
| Admission alerts | `/api/admissions/alerts` — already public | none |
| College static facts | `lib/data.js` + compare profiles | small JSON export |
| Connect handoff | deep link `?college=<slug>` | none — site handles availability |
| Lead alerts | Notify service — already standalone + multi-tenant | none, new API key |

Nightly ETL pull, no runtime coupling. If Campus Critique is down, the bot still works off its
snapshot. **Bonus: the nightly job doubles as a keep-alive** so the free Supabase project never
auto-pauses.

**Also an unplanned compliance win:** minors' chat data — phone numbers, family income, exam scores,
distress flags — lives in an isolated database with its own retention and purge path. When DPDP
parental-consent enforcement lands (May 2027), that is a far easier position to defend than
untangling it from production.

⚠️ Supabase allows **2 free projects per account**. Prod + dev are taken, so a third needs Pro
(~₹2,200/mo) or a free org under another founder's account.

## Still open

- **Persona / name.** "A senior who's been through it" is the right register. Does it get a name, or
  is it plainly Campus Critique? *(Leaning: no name — a named mascot invites "are you a real person?"
  and we've committed to never pretending. "Campus Critique" as a knowledgeable senior is cleaner.)*
