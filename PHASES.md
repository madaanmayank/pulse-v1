# Pulse — Phases

Four phases. The difference is not how much data we hold — it is what Pulse does with it.

| Phase | Pulse | The waiter | Test of success |
|---|---|---|---|
| **1 · Know** | Shows who is at the table | Reads it and decides | The waiter recognises the guest |
| **2 · Act** | Says what to do about it | Taps it done | The waiter acts without interpreting |
| **3 · See live** | Sees the phone in their hand | Intervenes at the right moment | Pulse spots it before the guest asks |
| **4 · Deepen** | Learns whether it worked | Is trusted more each shift | Recommendations improve on their own |


## Table 1 — Phase 1 · Know

**Guest intelligence.** Who is at the table. Everything here is display — Pulse shows what it knows and the waiter decides what it means.

*10 features · 9 live in the prototype*

| | Feature | Where it appears in the UI | Owner | Support | Notes |
|---|---|---|---|---|---|
| ✅ | All existing POS functionality in the premium theme | Table grid, allocate sheet, order items, void, payment summary | Pulse | — | — |
| ✅ | Guest name, masked phone, birthday | Guest card in the table view | Data | — | Static data from CRM |
| ✅ | New vs returning, visit count, days since last visit | Stat row above the guest card | Data | — | — |
| ✅ | Usual spend | Stat row | Data | — | — |
| ✅ | AI guest summary — history, ratings, comments and sentiment | Pulse Summary and Guest Summary cards | Data | — | — |
| ✅ | Known favourites and reorder, one combined section | Dish Recommendation strip | Data | — | — |
| ✅ | AI recommendations — 2–3 items, compact | Dish Recommendation strip | Data | — | — |
| ✅ | Top taste tags | Taste chips on the guest card | Data | — | — |
| ✅ | Network customer score | Score chip on the guest card | Data | — | — |
| ◻️ | Pre-shift briefing | — | Data | — | Same APIs, called earlier in the shift |

## Table 2 — Phase 2 · Act

**Action layer.** The same information becomes a checklist. Scoring, bucketing, service recovery, call waiter and a table sitting without ordering each turn into one thing to do.

*13 features · 10 live in the prototype*

| | Feature | Where it appears in the UI | Owner | Support | Notes |
|---|---|---|---|---|---|
| ✅ | Action framework — signal, why it matters, what to do, done | WHAT TO DO checklist at the top of the table view | Pulse | — | — |
| ✅ | Call waiter | Floor alert bar, red strip on the card, top row of the checklist | SW | Pulse | — |
| ✅ | Customer scoring and guest bucketing | Score chip, bucket label on the guest card and on the floor card | Data | Pulse | — |
| ✅ | Service recovery from past feedback **·Hero** | Red checklist row: prioritise this table | Data | Pulse | — |
| ✅ | Table taking no action — seated without ordering | Amber checklist row at 20 minutes | Data | Pulse | Timing computed in Pulse |
| ✅ | Recommend a dish for a stated preference | Checklist row naming the dish and the reason | Data | Pulse | — |
| ✅ | Recommended next-visit voucher **·Hero** | Checklist row: offer the voucher | CRM | Data, Pulse | — |
| ✅ | Remember this — staff teach the profile | Two taps and a line on the guest card | Pulse | Data | — |
| ✅ | Section-wise table selection | Section picker in the top bar, combines with the filters | Pulse | — | — |
| ✅ | Light theme **·Foundation** | Settings, Look | Pulse | — | — |
| ◻️ | Dynamic QR handling **·Foundation** | — | SW | — | Not a Pulse dashboard surface — SmartWeb side |
| ◻️ | Reservation integration (SevenRooms) **·Foundation** | Reservation list and seating a booked guest | Pulse | Data, SW | — |
| ◻️ | Pulse for Pro and Pulse for VIBE | — | Pulse | — | — |

## Table 3 — Phase 3 · See live

**QR journey.** What the guest is doing on their phone right now, plus the statics CRM owns — promotions, occasions, staff notes.

*15 features · 6 live in the prototype*

| | Feature | Where it appears in the UI | Owner | Support | Notes |
|---|---|---|---|---|---|
| ✅ | Help ordering — guest is stuck in the menu | Amber checklist row: walk them through the menu | SW | Data, Pulse | — |
| ✅ | Cart left too long — items added, never sent | Amber checklist row: ask if they meant to send it | SW | Data, Pulse | — |
| ✅ | Live browsing and repeated item views | Amber checklist rows naming the section or dish | SW | Data, Pulse | — |
| ✅ | Available promotions | Promotions card on the guest column, plus a checklist row to mention it | CRM | Data, Pulse | Staff informs the guest — Pulse cannot apply it |
| ✅ | Occasion and staff notes | Occasion chip and Staff Notes card | CRM | SW, Pulse | Static data from CRM |
| ✅ | Table state labels — at risk, lapsed, high value | Bucket label on the floor card | Data | Pulse | — |
| ◻️ | Restaurant and dish feedback collection | Rating shown on the profile once collected | SW | Data, Pulse | — |
| ◻️ | Preferred table | Shown when allocating a table and when taking a reservation | Data | Pulse | — |
| ◻️ | Staff notes after the visit | Prompt on closing the table | Data | Pulse | Collection should sit with CRM |
| ◻️ | Slipping regulars list **·Hero** | — | Data | Pulse | Open question: what is the staff action? |
| ◻️ | MunchMate target history | — | CRM | Data, Pulse | — |
| ◻️ | Vouchers and promos beyond the above | — | CRM | Pulse | Only if a merchant asks |
| ◻️ | Table layout management and merging **·Foundation** | — | Pulse | — | — |
| ◻️ | Ordering through Pulse directly **·Foundation** | — | Pulse | — | — |
| ◻️ | Order editing for POS-integrated merchants **·Foundation** | — | Pulse | — | — |

## Table 4 — Phase 4 · Deepen

**Depth.** Course timing, allergies, refill prediction and the learning loop. Every item needs a data route that does not exist yet.

*8 features · 0 live in the prototype*

| | Feature | Where it appears in the UI | Owner | Support | Notes |
|---|---|---|---|---|---|
| ◻️ | Course gap timer — time since the last course | — | Data | Pulse | Computed in Pulse |
| ◻️ | Allergies and dislikes | — | Data | SW, Pulse | Needs a collection route first |
| ◻️ | Drink refill timing **·Hero** | — | Data | Pulse, SW | — |
| ◻️ | Action log — recommended versus what staff did | — | Pulse | Data | — |
| ◻️ | Visit frequency trend | — | Data | Pulse | — |
| ◻️ | Network visits across branches | — | Data | Pulse | — |
| ◻️ | Preferred table auto-suggest | — | Data | SW, Pulse | — |
| ◻️ | Own reservation module | — | Pulse | Data, SW | — |

---

## Removed from the plan

| Item | Why |
|---|---|
| Last visit details | Not important right now |
| Multi-guest handling | Already covered by the party avatars in Phase 1 |
| Reorder and recommendations split by category | Not needed — one combined strip is enough |
| QR feedback sentiment as its own line | Folded into the AI guest summary |

✅ live in the prototype  ◻️ specified, not built
