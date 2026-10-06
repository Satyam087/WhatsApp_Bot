# Data strategy — JEE integration vs. LLM + RAG

Answering: *"research about JEE — if that can be integrated, or is LLM RAG enough?"*

**Short answer: neither alone. They solve different problems, and using RAG for the JEE part would
produce exactly the kind of confident wrong answer that ends the brand.**

---

## 1. Why "just use RAG" fails here

RAG retrieves text by **semantic similarity**. That works beautifully for prose and fails badly for
numeric tables.

Concretely: a student asks *"78 percentile se kya mil sakta hai?"* Under RAG, we'd embed 589,000
rows of JoSAA closing ranks and retrieve the "most similar" chunks. But embeddings have no concept
of **ordering** — nothing in the vector space encodes that closing rank 12,847 is *below* 13,002.
You get back chunks that *look* relevant and are numerically wrong, and the LLM presents them
fluently and with total confidence.

The failure mode isn't "the bot says I don't know." It's **"78 percentile pe aapko NIT Jalandhar
mil sakta hai"** — stated warmly, in Hinglish, and false. A student reorganises their whole plan
around it and misses real deadlines.

This is the single highest-hallucination-risk surface in the entire product, and it's also the one
where being wrong causes the most concrete harm. So it doesn't go anywhere near the LLM.

**The rule: if the answer is a number that can be looked up or computed, it must be looked up or
computed. RAG is for prose.**

---

## 2. The three-layer model

Not one retrieval system. Three, picked by the shape of the question.

| Layer | Handles | Mechanism | Hallucination risk |
|---|---|---|---|
| **A — Compute** | percentile → rank, loan eligibility, EMI, 4-year total cost | **Pure arithmetic in code** | **Zero** |
| **B — Structured lookup** | JoSAA/CSAB cutoffs, our 8 colleges' facts, fees, exam dates | **SQL over Postgres** | **Zero** |
| **C — RAG** | "Vedam ka hostel kaisa hai?", review prose, campus-life texture, blog content | **Embeddings over review text** | Low, and bounded |

Layer C is where RAG genuinely earns its place — thousands of words of verified student review prose
that no query language can interrogate. That's our actual moat, and it's unstructured, and RAG is
exactly right for it.

### Layer A is nearly free and disproportionately valuable

Percentile → rank is a published, deterministic formula:

```
Rank ≈ (100 − Percentile) × Total candidates appeared ÷ 100
```

For Aarav from the sample conversation — 78.4 percentile, ~14.5 lakh candidates:

```
(100 − 78.4) × 14,50,000 ÷ 100  ≈  rank 3,13,200
```

Against **67,323 total JoSAA seats**, that single computed line tells him more than an hour of
YouTube. It costs us **one multiplication**, needs no API, no scraping, no vendor — just the year's
appeared-candidate count as a constant.

Caveat we must state in the output: NTA's own guidance is that this is indicative. Marks→percentile
varies with shift difficulty and normalisation; percentile→rank is the exact part. So the bot says
**"roughly 3.1 lakh rank"**, never "your rank is 313,200."

The same layer runs the loan math from Stage 4 — `max_loan ≈ 4 × annual_income`, EMI, and the
4-year all-in total. All arithmetic. All exact. **None of it goes near a model.**

---

## 3. Can JEE/JoSAA data actually be integrated? Yes, and it's free

| Source | What it gives | Access | Cost |
|---|---|---|---|
| **JoSAA official archive** (`josaa.admissions.nic.in` → opening/closing rank archive) | Round-wise OR/CR for 23 IITs, 31 NITs, 26 IIITs, 56 GFTIs | Public web tables, no login | ₹0 |
| **CSAB** | Special-round cutoffs | Public | ₹0 |
| **`Harith-Y/JoSAA-CSAB-Closing-Rank-Predictor`** (GitHub) | 588,619 rows, 2016–2026 R1–R5, + CSAB ~55k rows, + Playwright scrapers | Public repo | ₹0 |
| State counselling (UPTAC, MHT-CET, KCET, WBJEE, …) | State-quota cutoffs | **~15+ separate portals**, different formats | ₹0 but high effort |

**⚠️ Licensing caution on the GitHub dataset.** That repo declares **no license**. No license means
no grant of rights — we cannot safely ship its CSVs in a commercial product. Use it as a *reference
implementation* for the scraper (it's good work, and it proves the shape of the data), but pull the
actual data ourselves from the official JoSAA archive, which is public government data. Roughly
4 hours of scraping, once a year.

**State counselling is out of scope, probably permanently.** Fifteen-plus portals, each with its own
format and schedule, to serve a branch of the option map we only need to *point at*. The bot's
correct behaviour is: *"UP domicile means 85% of AKTU seats are reserved for you — that's a real
advantage. I don't have verified UPTAC cutoff data, so go to uptac.samarth.edu.in directly. Don't
take my word for it."* Honest, useful, and zero build.

---

## 4. Timing — and this changes the roadmap

**It is late September. The JoSAA 2026 cycle closed in July (Round 5 published 16 July 2026).**
The next set of cutoffs doesn't exist until roughly June 2027.

So the students who will message us *this month* are not June panic-decision students. They are:
- **Class 11 / 12 students planning early** — the best possible audience for deadline tracking (#10)
- **Current droppers** preparing for JEE 2027
- **Students who enrolled in July 2026 and are now 2–3 months in** ← this is the interesting one

That last group matters more than it looks. The peak counselling cohort is **~8 months away**, which
means two things:

1. **JoSAA integration is genuinely v1, not v0.** Building a scraper in week 1 for data that becomes
   relevant in June 2027 would be the wrong week's work. Ship Layer A (the arithmetic — which *is*
   immediately useful) and have the bot honestly point at official sources for cutoffs.

2. **The review flywheel can run first, and should.** Students who joined Vedam / Intellipaat / Alta
   in July are exactly 2–3 months in — the sweet spot where the experience is vivid and the honeymoon
   is over. Capability #12 (review collection over WhatsApp) was scoped as v1 because it seemed to
   depend on the counselling funnel. It doesn't. It can run *now*, against students we reach through
   the existing platform, and every review collected between now and June makes the counselling bot
   materially better for the cohort that actually matters.

**That inverts the roadmap in a useful way:** we have ~8 months of low-stakes runway to make the bot
genuinely good, while using that time to deepen the data moat that the bot will run on. Launching
into peak season with an untested bot would have been the real risk. We've accidentally got the
timing right.

---

## 5. Recommended scope

### v0 (week 1)
- **Layer A in full** — percentile→rank, loan ceiling, EMI, 4-year all-in. Pure code.
- **Layer B for our 8 colleges** — `wa_college_facts`, SQL, source + `updated_at` on every fact.
- **Layer C, thin** — RAG over existing approved review text. Start with Postgres `pgvector` in the
  Supabase instance we already pay nothing for. No separate vector DB.
- **JoSAA: honest pointer, no data.** "I don't have verified cutoff data — here's the official source."

### v1 (Nov–Jan, well before June)
- Scrape JoSAA + CSAB from the official archive into Layer B. One table, one annual job.
- Rank-band college prediction with **explicit confidence bands**, never point estimates.
- Deadline tracking (#10) wired to the existing admission-alert cron.
- Compare-on-demand (#14) over `collegeCompareProfiles`.

### Never (for now)
- State counselling cutoffs — point at official portals instead.
- Fine-tuning. Our data changes every admission cycle; a fine-tuned model bakes in facts that expire.
  RAG + SQL keeps the facts swappable, which is the whole point.

---

## 6. The guard that makes this safe

Layers A and B produce a **fact block** — every value carrying a `source` and `updated_at`. The
generation prompt may only phrase what's in that block. Then a cheap post-generation check scans the
output for any `₹` / `LPA` / `%` / rank figure that doesn't appear in the block, and on a hit falls
back to a templated response.

That guard is ~40 lines of code and it is the difference between a trustworthy product and a
confident liar. It runs on every single outbound message.

---

## Sources
- [JoSAA official opening/closing rank archive](https://josaa.admissions.nic.in/applicant/seatmatrix/openingclosingrankarchieve.aspx)
- [Harith-Y/JoSAA-CSAB-Closing-Rank-Predictor](https://github.com/Harith-Y/JoSAA-CSAB-Closing-Rank-Predictor) — 588,619 rows, 2016–2026, **no license declared**
- [Sbrjt/josaa-cutoffs](https://github.com/Sbrjt/josaa-cutoffs) · [Bhagya-Anand-18/edusearch-api](https://github.com/Bhagya-Anand-18/edusearch-api)
- [Career Point — percentile-to-rank method, JEE Main 2026](https://careerpoint.ac.in/blog/how-to-calculate-jee-main-2026-rank-from-nta-percentile-score-criteria-and-method/)
- [Motion — JoSAA 2026 opening/closing ranks, Round 5 out 16 July 2026](https://motion.ac.in/examinfo/josaa-opening-and-closing-ranks/)
- [Collegedunia — JoSAA seat matrix 2026 (67,323 seats)](https://collegedunia.com/exams/jee-main/seat-matrix)
- [UPTAC official counselling portal](https://uptac.samarth.edu.in/) · [Careers360 — UPTAC cutoff 2026](https://engineering.careers360.com/articles/uptac-cutoff)
