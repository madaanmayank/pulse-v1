# Pulse — Phases

Three phases. The difference is not how much data we hold — it is what Pulse does with it.

| Phase | Pulse | The waiter | Test of success |
|---|---|---|---|
| **1 · Know** | Shows who is at the table, and no more | Reads it and decides | The waiter recognises the guest |
| **2 · Act** | Says what to do about it | Ticks it done, or dismisses it | The waiter acts without interpreting |
| **3 · Anticipate** | Watches the phone and the clock | Gets there before the guest asks | Pulse spots it first, and nothing sits ignored |

> The prototype carries its own **Design guide** screen — every element, colour and state rendered from the product's own code. Open it from the bottom nav.


## Phase 1 · Know

**Guest intelligence.** Who is at the table. Everything here is display — Pulse shows what it knows and the waiter decides what it means.

*10 features · 9 live in the prototype · 0 actions the engine can raise*

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

## Phase 2 · Act

**Action layer.** The same information turned into a checklist. Scores, buckets, recovery, guest requests and a table sitting without ordering all become one thing to do.

*17 features · 14 live in the prototype · 10 actions the engine can raise*

| | Feature | Where it appears in the UI | Owner | Support | Notes |
|---|---|---|---|---|---|
| ✅ | Action framework — signal, why it matters, what to do, done | WHAT TO DO checklist at the top of the table view | Pulse | — | — |
| ✅ | Guest requests — five fixed kinds | Capsule row above the floor, one per table with its kind and wait; red strip on the card; top of the checklist; and a waiting-elsewhere bar inside any open table | SW | Pulse | Concern, question, waiter, water, cutlery. No free text on the guest side, so Pulse never shows a description |
| ✅ | Action lifecycle — new, unattended, done or dismissed **·Hero** | NEW flag on an action raised mid-meal, WAITING n MIN once nobody has closed it for 15, an Unattended group in the capsule row, and a dismiss × on every row | Pulse | — | Answers the two ways an action gets lost: unnoticed when it appears, or ignored after |
| ✅ | Manager view — only what is actually a manager’s job | Manager tab: a guest is unhappy · needs you at the table · nobody has picked these up · waiting on your approval · a roster of the rest | Pulse | Data | Water, cutlery, questions and "called the waiter" are errands and never reach this screen. Every row carries a decision, never an Open table link |
| ✅ | Comp and voucher approvals | Gold approve buttons on the manager card for the gesture Pulse suggests, reversible for five seconds | Pulse | CRM | — |
| ✅ | On the house — the manager’s own call **·Hero** | A dashed control on every manager card opens six things they can send: dessert, a round, champagne, a starter, coffee, or a next-visit voucher | Pulse | CRM | Pulse suggests a gesture; the manager decides. The sheet names the occasion and any concern so the judgement is informed |
| ✅ | Customer scoring and guest bucketing | Score chip, bucket label on the guest card and on the floor card | Data | Pulse | — |
| ✅ | Service recovery from past feedback **·Hero** | Red checklist row: prioritise this table | Data | Pulse | — |
| ✅ | Table taking no action — seated without ordering | Amber checklist row at 20 minutes | Data | Pulse | Timing computed in Pulse. Phase 3 replaces it with something sharper once we can see the phone |
| ✅ | Recommend a dish for a stated preference | Checklist row naming the dish and the reason | Data | Pulse | — |
| ✅ | Recommended next-visit voucher **·Hero** | Checklist row: offer the voucher | CRM | Data, Pulse | — |
| ✅ | Remember this — staff teach the profile | Two taps and a line on the guest card | Pulse | Data | — |
| ✅ | Section-wise table selection | Section picker in the top bar, combines with the filters | Pulse | — | — |
| ✅ | Light theme **·Foundation** | Settings, Look | Pulse | — | — |
| ◻️ | Dynamic QR handling **·Foundation** | — | SW | — | Not a Pulse dashboard surface — SmartWeb side |
| ◻️ | Reservation integration (SevenRooms) **·Foundation** | Reservation list and seating a booked guest | Pulse | Data, SW | — |
| ◻️ | Pulse for Pro and Pulse for VIBE | — | Pulse | — | — |

## Phase 3 · Anticipate

**Live behaviour and timing.** The guest’s phone and the clock. A cart built but never sent, a menu open too long, a table nobody has touched, mains going cold, glasses empty — and the calls that are the manager’s to make.

*25 features · 12 live in the prototype · 12 actions the engine can raise*

| | Feature | Where it appears in the UI | Owner | Support | Notes |
|---|---|---|---|---|---|
| ✅ | Cart built but never placed **·Hero** | Red checklist row naming the item count and how long it has sat unsent | SW | Data, Pulse | An order the kitchen has never seen — the highest-value thing on this list |
| ✅ | Menu open too long with nothing added | Amber row: go and help them choose, with minutes in the menu and dishes opened | SW | Data, Pulse | — |
| ✅ | Table nobody has started — menu never opened | Amber row: go and start them off | SW | Data, Pulse | — |
| ✅ | One dish opened again and again | Amber row naming the dish and the view count | SW | Data, Pulse | — |
| ✅ | Lingering in one part of the menu | Amber row: offer to choose it with them | SW | Data, Pulse | — |
| ✅ | Order gap timer — time since anything was last ordered | Amber row: offer the next course, naming the minutes since the last order | Data | Pulse | Order data only. Pulse cannot see whether a plate is on the table, so it does not claim to |
| ✅ | Drink reorder timing **·Hero** | Amber row: offer another round, naming the minutes since the last drink was ordered | Data | Pulse, SW | Order data only — not glass level |
| ✅ | Managerial touchpoints | Visit the table — on the manager card and as a Manager row in the checklist | Pulse | Data | Judgement, not errands, so it only appears once the live layer is on |
| ✅ | Available promotions | Promotions card on the guest column, plus a checklist row to mention it | CRM | Data, Pulse | Staff informs the guest — Pulse cannot apply it |
| ✅ | Occasion and staff notes | Occasion chip and Staff Notes card | CRM | SW, Pulse | Static data from CRM |
| ✅ | Table state labels — at risk, lapsed, high value | Bucket label on the floor card | Data | Pulse | — |
| ◻️ | Restaurant and dish feedback collection | Rating shown on the profile once collected | SW | Data, Pulse | — |
| ◻️ | Preferred table | Shown when allocating a table and when taking a reservation | Data | Pulse | — |
| ✅ | Staff notes after the visit | Optional note sheet when closing a table — a category and one line, saved to the guest | Data | Pulse | Skippable by design. Storage should sit with CRM |
| ◻️ | Allergies and dislikes | — | Data | SW, Pulse | Needs a collection route first |
| ◻️ | Action log — recommended versus what staff did | — | Pulse | Data | The dismiss reason is the missing half of this |
| ◻️ | Visit frequency trend | — | Data | Pulse | — |
| ◻️ | Network visits across branches | — | Data | Pulse | — |
| ◻️ | Slipping regulars list **·Hero** | — | Data | Pulse | Open question: what is the staff action? |
| ◻️ | MunchMate target history | — | CRM | Data, Pulse | — |
| ◻️ | Vouchers and promos beyond the above | — | CRM | Pulse | Only if a merchant asks |
| ◻️ | Table layout management and merging **·Foundation** | — | Pulse | — | — |
| ◻️ | Ordering through Pulse directly **·Foundation** | — | Pulse | — | — |
| ◻️ | Order editing for POS-integrated merchants **·Foundation** | — | Pulse | — | — |
| ◻️ | Own reservation module | — | Pulse | Data, SW | — |

---

## What the guest can send

Five buttons on their phone and no free-text field — so Pulse never shows a description the guest could not type. A table can have several outstanding at once; they collect on one capsule, because one trip handles them all, and the wait counts from the first ask.

| Request | What the waiter does | Weight |
|---|---|---|
| Raised a concern | Hear them out | A guest is unhappy and cannot say why — red |
| Has a question | Answer their question | A guest is stuck and needs a person — violet |
| Called the waiter | Go to the table | Someone has to go over — gold |
| Asked for water | Take water over | Something to carry — teal |
| Asked for cutlery | Take cutlery over | Something to carry — sand |

A **concern** is the exception: it is the manager's to handle. The floor capsule and the card both say *manager*, and the waiter's own action is to go and get them.


## What happens to an action

An action gets lost two ways: nobody notices it arrive, or everybody ignores it afterwards. Both have a state.

| State | When | How it shows |
|---|---|---|
| New | Raised after the waiter had already looked at this table | Flag on the row, ring on the card badge |
| Open | Being worked | Plain row |
| Unattended | 15 min and nobody has closed it | Age chip on the row and the card, its own group in the floor row, and a section in the Manager tab |
| Done | Ticked — right-hand side of the row | Strikes through, holds three seconds with Undo in place, then collapses |
| Dismissed | Dismissed — left-hand side of the row | Same three-second grace, then gone and not raised again |

Only **work** ages. A relational cue — welcome them back, acknowledge the occasion — is true for the whole visit and never goes overdue.

At most **5 rows** show at once, work first. When nothing is open the section closes quietly to *All good here*.


## What the manager sees, and why

Errands never reach this screen. Water, cutlery, a question and "called the waiter" are the floor's job.

| Section | The rule |
|---|---|
| A concern, raised just now | They pressed “raise a concern” on their phone minutes ago. There is no detail — the app has no text field — so go and ask. |
| Last visit went badly | Poor feedback or a service failure the last time they came in. Tonight is the visit that fixes it. |
| Worth a word from you | Top spenders, regulars, and guests who have not been in for a while. A hello from the manager lands differently. |
| Nobody has picked these up | An action has been open ${STALE_MIN} minutes or more and no one has closed it. Chase the section, or take it on yourself. |
| Waiting on your approval | Comps and vouchers. Only you can authorise money off a bill. |

Every row carries a decision — *Handled*, *Visit the table*, *Chase the section*, *I will take it* — never an "open table" link, and never more than three. **On the house** is always available on every card: dessert, round of drinks, glass of champagne, starter, coffee and petits fours, voucher for the next visit. The sheet names the occasion and any concern, so the judgement is informed.


---

## Merged

| Was | Now | Why |
|---|---|---|
| Phase 3 "See live" and Phase 4 "Deepen" | One Phase 3 · Anticipate | Turning them on separately gave no distinct value. Live phone behaviour and the clock are the same job: get there before the guest has to ask. |

## Removed from the plan

| Item | Why |
|---|---|
| Last visit details | Not important right now |
| Multi-guest handling | Already covered by the party avatars in Phase 1 |
| Reorder and recommendations split by category | One combined strip is enough |
| QR feedback sentiment as its own pill | Folded into the guest summary — on its own it said nothing |
| "On my way" on a request | A waiter walking to a table does not stop to tell the tablet |
| Crown badge on the floor card | Competed with the action count. One indicator per card |
| Gold "just seated" card treatment | Made a table with nothing to do look louder than one with three things to do |
| "+2" beside the top action | The count badge already says how many. Two numbers for one fact |
| "Walk-in" as a table name | A table with no profile is a Guest |
| Per-action icons on unattended capsules | Seven glyphs nobody was taught. They all mean one thing |
| "The plates are sitting" / "glasses are empty" | We have order data, not service data. It says "last ordered 34 min ago" |
| Toast-based undo on a checklist row | Undo sits in the row for three seconds, where the thumb already is |
| Green "done" rows stacking under the open ones | They refilled the space you just cleared |
| Errands in the Manager tab | Water, cutlery and questions are the floor’s job |
| "Open table" as a manager action | Navigation is not a decision |
| The checklist at Phase 1 | Phase 1 is display only. An empty checklist there implied nothing needed doing |

✅ live in the prototype  ◻️ specified, not built
