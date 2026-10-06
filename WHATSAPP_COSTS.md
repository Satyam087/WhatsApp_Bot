# WhatsApp cost model — messaging only

No LLM, no infra. Just what Meta bills us.

**Rate card, effective 1 October 2026** (published; confirmed for +91):

| Category | India rate | Notes |
|---|---|---|
| **Service** (bot replies inside the 24h window) | **₹0.115** | **First 1,000/month per phone number free.** No volume discount. |
| Utility template | ₹0.115 | No free tier |
| Marketing template | **₹0.8631** | **7.5× service** |
| **Inbound** (anything the student sends) | **₹0.00** | Always free |
| **72h Free Entry Point** (Click-to-WhatsApp ads) | **₹0.00** | Unchanged by the Oct 1 change |

Rates are pre-tax. **+18% GST** on Meta's India billing — reclaimable as input tax credit if
GST-registered (which `connect_prd_v2.md` already recommends). Figures below are ex-GST unless stated.

We're on **Cloud API direct**, so these are the *only* messaging numbers. A BSP would add ~26% on
top (₹0.145 service, ₹1.09 marketing) plus ₹1,500–3,200/month subscription.

---

## 1. The Aarav conversation

**11 outbound messages. 11 inbound (free).**

| Situation | Cost |
|---|---|
| Inside the monthly free 1,000 | **₹0.00** |
| Student arrived via a Click-to-WhatsApp ad | **₹0.00** |
| Normal, billed | **₹1.27** (11 × ₹0.115) |
| …with GST | ₹1.49 |

The 3-week follow-up sits outside the 24h window, so it needs a template:
**₹0.115 if Meta classifies it utility — ₹0.8631 if marketing.** See §5.

---

## 2. What an "average" conversation actually is

The 11-message ideal is not the average. Real distribution:

| Segment | Share | Outbound | Weighted |
|---|---:|---:|---:|
| Bounce — never replies | 25% | 2 | 0.50 |
| Drops during intake | 25% | 5 | 1.25 |
| Completes all 7 stages | 35% | 12 | 4.20 |
| **Deep — many follow-up questions** | **15%** | **30** | **4.50** |
| | | **≈ 10.5** | **outbound/user** |

> **★ The 15% who engage deeply generate 43% of all messages.**
> Our best users are our most expensive users. That's the number to manage — see §6.

---

## 3. Monthly cost by volume

Three design scenarios. "Chatty" is what we get if we *don't* batch questions and use tap-buttons —
i.e. a free-running LLM agent that replies to everything conversationally.

| Conversations/mo | Lean (8 msgs) | **Base (10.5)** | Chatty (20) |
|---:|---:|---:|---:|
| **1,000** | ₹805 | **₹1,081** | ₹2,185 |
| **5,000** | ₹4,485 | **₹5,865** | ₹11,385 |
| **10,000** | ₹9,085 | **₹11,845** | ₹22,885 |
| *per user* | *₹0.81–0.91* | ***₹1.08–1.18*** | *₹2.19–2.29* |

**Design discipline is worth ~2×.** Batching three questions into one message and using tap-buttons
instead of free-text prompts is the difference between ₹0.91 and ₹2.29 per user. Same conversation,
same outcome for the student.

The free 1,000/month matters at pilot scale (it covers ~95 full conversations) and is rounding error
above 1,000 users.

---

## 4. The CTWA lever — the biggest one

Students arriving from a Click-to-WhatsApp ad open a **72-hour window where everything is free**,
and that survived the Oct 1 change.

| Conversations/mo | 0% CTWA | 30% CTWA | 60% CTWA | 100% CTWA |
|---:|---:|---:|---:|---:|
| 1,000 | ₹1,081 | ₹722 | ₹363 | ₹0 |
| 5,000 | ₹5,865 | ₹4,071 | ₹2,277 | ₹0 |
| **10,000** | **₹11,845** | **₹8,257** | **₹4,669** | **₹0** |

Ad spend is separate (₹8–25/click in this category) — but that's acquisition budget you'd spend
anyway, and it zeroes the messaging line. **Paid acquisition and cheapest messaging are the same
channel.** This should influence how we launch, not just how we budget.

---

## 5. Follow-ups — the categorisation risk

Anything outside the 24h window needs a template, and **templates get no free tier**.

| Conversations/mo | 1 utility each | 1 marketing each | Difference |
|---:|---:|---:|---:|
| 1,000 | ₹115 | ₹863 | ₹748 |
| 5,000 | ₹575 | ₹4,316 | ₹3,741 |
| **10,000** | **₹1,150** | **₹8,631** | **₹7,481** |

**Meta decides the category, not us.** Best read:
- **"Your VSAT deadline is in 6 days"** → opted-in, specific, factual → should pass as **utility**
- **"Hey, how did your college decision go?"** → open-ended check-in → likely **marketing**

So the 3-week follow-up in the sample conversation is the risky one. Two mitigations, both free:
1. **Anchor the follow-up to a concrete event** the student asked to be tracked — a deadline, an
   exam date, a result. That's utility-shaped, and it's also a better message.
2. **Get them to message first.** A follow-up that prompts a reply reopens the 24h service window,
   and everything after it is service-rate.

⚠️ Also: business-initiated messages are subject to **messaging tier limits** (1K → 10K → 100K →
unlimited per 24h). At 10,000 follow-ups/month (~333/day) we need at least the 1K tier, which
requires business verification. Not a cost, but a launch dependency.

---

## 6. All-in monthly WhatsApp bill

Base case: 10.5 messages/conversation + one utility follow-up each, 0% CTWA.

| Conversations/mo | Conversation | Follow-up | **Total** | With GST | Per user |
|---:|---:|---:|---:|---:|---:|
| **1,000** | ₹1,081 | ₹115 | **₹1,196** | ₹1,411 | ₹1.20 |
| **5,000** | ₹5,865 | ₹575 | **₹6,440** | ₹7,599 | ₹1.29 |
| **10,000** | ₹11,845 | ₹1,150 | **₹12,995** | ₹15,334 | ₹1.30 |

### The headline

**At 10,000 conversations a month, WhatsApp costs us about ₹13,000 — roughly ₹1.30 per student
counselled.** Even the worst realistic case (chatty design, all-marketing follow-ups, no CTWA) lands
near ₹31,500/month.

**Messaging is not what will break this budget.** It's an order of magnitude cheaper than one
counsellor's salary, and one referred admission in this category (₹20–60k) covers a month of it.
The LLM bill (~₹1.20/conversation) is now *comparable to* the messaging bill — so if we optimise
anything later, optimise both or neither.

---

## 7. Cost control — in priority order

| # | Lever | Impact | Cost to do |
|---|---|---|---|
| 1 | **Drive acquisition through Click-to-WhatsApp ads** | up to **−100%** | ad spend you'd spend anyway |
| 2 | **Batch questions; tap-buttons over free text** | **−55%** vs chatty | design discipline only |
| 3 | ~~Cap the deep tail~~ **— withdrawn, see below** | — | — |
| 4 | **4-second debounce** — 3 rapid messages get 1 reply | −10–15% | already in the architecture |
| 5 | **Anchor follow-ups to a tracked event** so they stay utility | avoids a 7.5× hit | free |
| 6 | Stay on Cloud API direct, not a BSP | −26% + no subscription | already decided |

### ⚠️ Lever 3 withdrawn — there is no upper limit, and there shouldn't be

I originally recommended capping engaged students at ~6 follow-up questions. **That was wrong**, and
the absolute numbers say so:

| One student's lifetime | Outbound | Cost |
|---|---:|---:|
| Full 7-stage conversation | 11 | ₹1.27 |
| …+ heavy clarification, parent joins, deep Q&A | 54 | ₹6.21 |
| …+ returns across a 4–6 week decision | 79 | ₹9.09 |
| …+ deadline tracking through admission | 89 | ₹10.24 |
| **…+ post-enrolment reviews at 3/6/12 months** | **104** | **₹11.96** |
| Absolute pathological worst case | 500 | ₹57.50 |

**A power user we hold by the hand for twelve months costs about ₹12.** One Connect booking nets
₹19.80 — which buys 172 messages. One referred admission in this category (₹20k–60k) is worth
**170,000 to 520,000 messages.**

Capping a deeply engaged student to save ₹2 is optimising a rounding error, and it would cap exactly
the students most likely to book a Connect call, submit a review, and send us their friends — the
three things the whole model depends on. **No message cap. The student gets as many as they need.**

What survives from lever 3 is narrower and still true: **link to the college page when the page is
genuinely the better answer** — a full fee table reads better as a table than as eight chat bubbles.
Do it for the student's sake, never to save ₹0.92.

Levers 1, 2 and 4 stand — but note they're all *design* levers that make the bot better to use
anyway. None of them involve serving a student less.

---

## Sources
- [ChatMaxima — WhatsApp service message pricing from 1 Oct 2026, by country and currency](https://chatmaxima.com/blog/whatsapp-service-message-pricing-october-2026/)
- [Mark360 — WhatsApp service message pricing India, 1 Oct 2026 (₹0.115)](https://mark360.ai/blog/whatsapp-service-message-pricing-october-1-2026)
- [Meta — upcoming pricing updates for service and utility messages](https://developers.facebook.com/documentation/business-messaging/whatsapp/pricing/non-template-messages)
- [360dialog — service message charging starts 1 Oct 2026](https://360dialog.com/blog/whatsapp-service-message-charging-october-2026/)
- [Chatbotscape — CTWA and the 72-hour free entry point](https://chatbotscape.com/glossary/click-to-whatsapp-ads)
- [AiSensy — India rate card 2026 (BSP-marked rates for comparison)](https://aisensy.com/pricing)
