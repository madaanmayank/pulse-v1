# Pulse — Phases

Three phases. The difference is not how much data we hold — it is what Pulse does with it.

| Phase | Pulse | The waiter | Test of success |
|---|---|---|---|
| **1 · Know** | Shows who is at the table, and no more | Reads it and decides | The waiter recognises the guest |
| **2 · Act** | Says what to do about it | Ticks it done, or dismisses it | The waiter acts without interpreting |
| **3 · Anticipate** | Watches the phone and the clock | Gets there before the guest asks | Pulse spots it first, and nothing sits ignored |

> The prototype carries its own **Design guide** screen, reachable from the bottom nav and split in two.
>
> **Part one** takes each phase in turn — what it puts on screen, and every action it can raise with the weight and the limit on each. The block for the phase you are running is marked *Running now*.
>
> **Part two** is the reference that applies to all three: urgency levels, the action lifecycle, an annotated live card, table states, the screens, the palette, and the rules.
>
> Every element, colour, state and limit on that page is rendered from the product's own code, so it cannot describe a rule that is no longer true.


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

*19 features · 16 live in the prototype · 7 actions the engine can raise*

| | Feature | Where it appears in the UI | Owner | Support | Notes |
|---|---|---|---|---|---|
| ✅ | Action framework — signal, why it matters, what to do, done | WHAT TO DO checklist at the top of the table view | Pulse | — | — |
| ✅ | Guest requests — five fixed kinds | Capsule row above the floor, one per table with its kind and wait; red strip on the card; top of the checklist; and a waiting-elsewhere bar inside any open table | SW | Pulse | Concern, question, waiter, water, cutlery. No free text on the guest side, so Pulse never shows a description |
| ✅ | Action lifecycle — new, unattended, done or dismissed **·Hero** | New flag on an action raised mid-meal, an age chip once it passes its own limit, an Overdue group in the floor row, and a dismiss × on every row | Pulse | — | Answers the two ways an action gets lost: unnoticed when it appears, or ignored after. Each kind has its own patience |
| ✅ | Manager view — only what is actually a manager’s job | Manager tab: a guest is unhappy · needs you at the table · nobody has picked these up · waiting on your approval · a roster of the rest | Pulse | Data | Water, cutlery, questions and "called the waiter" are errands and never reach this screen. Every row carries a decision, never an Open table link |
| ✅ | Comp and voucher approvals | Gold approve buttons on the manager card for the gesture Pulse suggests, reversible for five seconds | Pulse | CRM | — |
| ✅ | On the house — the manager’s own call **·Hero** | A dashed control on every manager card opens six things they can send: dessert, a round, champagne, a starter, coffee, or a next-visit voucher | Pulse | CRM | Pulse suggests a gesture; the manager decides. The sheet names the occasion and any concern so the judgement is informed |
| ✅ | Customer scoring and guest bucketing | Score chip, bucket label on the guest card and on the floor card | Data | Pulse | — |
| ✅ | Service recovery from past feedback **·Hero** | Red checklist row: prioritise this table | Data | Pulse | — |
| ✅ | Table taking no action — seated without ordering | Amber checklist row at 20 minutes | Data | Pulse | Timing computed in Pulse. Phase 3 replaces it with something sharper once we can see the phone |
| ✅ | One greeting and one suggestion, not four rows | — | Pulse | Data | — |
| ✅ | The opening ten minutes | Amber row on a freshly seated table — pour water and hand them menus — plus a Just seated group in the floor row counting down the window. Retires when they order | Pulse | Data | Chased after 8 minutes. The first ten decide the visit |
| ✅ | One floor row for everything time-critical **·Hero** | Guests waiting · Overdue · Just seated — three groups of capsules above the grid, each capped so the row never becomes a wall | Pulse | — | Wraps rather than scrolling a group off the edge, because a group you cannot see is a group nobody works |
| ✅ | Per-request patience **·Hero** | Each request kind carries its own limit before Pulse says nobody picked it up — concern 3 min, question and waiter 6, water and cutlery 10 | Pulse | — | Fifteen minutes for cutlery is defensible; fifteen for an unhappy guest is not |
| ✅ | Remember this — staff teach the profile | Two taps and a line on the guest card | Pulse | Data | — |
| ✅ | Section-wise table selection | Section picker in the top bar, combines with the filters | Pulse | — | — |
| ✅ | Light theme **·Foundation** | Settings, Look | Pulse | — | — |
| ◻️ | Dynamic QR handling **·Foundation** | — | SW | — | Not a Pulse dashboard surface — SmartWeb side |
| ◻️ | Reservation integration (SevenRooms) **·Foundation** | Reservation list and seating a booked guest | Pulse | Data, SW | — |
| ◻️ | Pulse for Pro and Pulse for VIBE | — | Pulse | — | — |

## Phase 3 · Anticipate

**Live behaviour and timing.** The guest’s phone and the clock. A cart built but never sent, a menu open with nothing added, a table nobody has started, a long gap since anything was ordered — and the calls that are the manager’s to make.

*26 features · 13 live in the prototype · 13 actions the engine can raise*

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
| ✅ | Recommended next-visit voucher **·Hero** | Checklist row: offer the voucher | CRM | Data, Pulse | A next-visit play, not a tonight play |
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

## The floor row

Three groups above the grid, read left to right, each capped at 4 capsules. It wraps rather than scrolling a group off the edge — a group you cannot see is a group nobody works.

| Group | What is in it |
|---|---|
| **Guests waiting** | They pressed a button on their phone. Coloured by kind, with a crown where the manager owns it |
| **Overdue** | Past its limit with nobody closing it. Longest first, then **+N more** into the Manager tab |
| **Just seated** | Sat down within the last 10 minutes and has not ordered. Counts down the window |

## Every action, and when Pulse chases it

Each phase adds actions, and each carries its own patience. **Gold is something to say, so it never goes overdue.** A guest request is timed from the moment they pressed the button; everything else from when Pulse raised it.


### Phase 2 · Act — 7 actions

| What it is | What the waiter does | Weight | Chased after |
|---|---|---|---|
| Last visit went wrong | Check on them twice as often | Intervene | 15 min |
| A guest request | Depends on the request | Intervene | 3–10 min |
| The opening ten minutes | Pour water and hand them menus | A task | 8 min |
| Seated, nothing ordered | Go and take their order | A task | 15 min |
| On the table a long time | Offer the check | A task | 25 min |
| A returning guest | Greet them by their first name | Say it | never |
| A taste we know about | Recommend the closest dish to their taste, or lead with their usual | Say it | never |

### Phase 3 · Anticipate — 13 actions

| What it is | What the waiter does | Weight | Chased after |
|---|---|---|---|
| Cart built, order never placed | Ask them to send their order | Intervene | 15 min |
| High-value guest let down last time | Visit the table | Intervene | 15 min |
| A concern, for the manager | Visit the table | Intervene | 15 min |
| Menu never opened | Take their order in person | A task | 15 min |
| Long in the menu, nothing added | Talk them through the menu | A task | 15 min |
| A long gap since anything was ordered | Offer the next course | A task | 15 min |
| A long gap since a drink was ordered | Offer another round | A task | 15 min |
| One dish opened again and again | Talk them through that dish | A task | 15 min |
| Lingering in one part of the menu | Offer to choose it with them | A task | 15 min |
| An occasion | Mention the occasion | Say it | never |
| A next-visit voucher | Offer the voucher | Say it | never |
| A promotion they qualify for | Mention the promotion | Say it | never |
| A top spender | Visit the table | Say it | never |

## What the guest can send

Five buttons on their phone and no free-text field — so Pulse never shows a description the guest could not type. A table can have several outstanding at once; they collect on one capsule, because one trip handles them all, and the wait counts from the first ask.

| Request | What the waiter does | Chased after | Weight |
|---|---|---|---|
| Raised a concern | Hear them out | **3 min** | A guest is unhappy and cannot say why — red |
| Has a question | Answer their question | **6 min** | A guest is stuck and needs a person — violet |
| Called the waiter | Go to the table | **6 min** | Someone has to go over — gold |
| Asked for water | Take water over | **10 min** | Something to carry — teal |
| Asked for cutlery | Take cutlery over | **10 min** | Something to carry — sand |

A **concern** is the exception on ownership: it is the manager's to handle. The floor capsule and the card both say *manager*, and the waiter's own action is to go and get them.


## What happens to an action

| State | When | How it shows |
|---|---|---|
| New | Raised after the waiter had already looked at this table | Flag on the row, ring on the card badge |
| Open | Being worked | Plain row |
| Overdue | Past its own limit with nobody closing it | Age chip on the row and the card, an Overdue group in the floor row, and a section in the Manager tab |
| Done | Ticked — right of the row | Strikes through, holds three seconds with Undo in place, then collapses |
| Dismissed | Dismissed — left of the row | Same three-second grace, then gone and not raised again |

At most **5 rows** show at once, work first. When nothing is open the section closes quietly to *All good here*.


## When an order arrives

Everything else Pulse reports is something that has *not* happened. This is the one piece of good news, and it changes what a waiter does next — stop walking over, start watching the pass.

| What happens | Why |
|---|---|
| A toast names the table, the item count and the value | Once, not repeatedly |
| An **ORDER IN** chip sits on the card | Until a waiter opens that table, or 6 minutes pass — whichever is first |
| The unsent cart clears | It went through, so nothing is sitting in it |
| Actions chasing that table for not ordering retire | The order is in; the nudge did its job |
| The order-gap clock restarts at zero | The next nudge is measured from this order, not the one before |

## What the manager sees, and why

Three sections, not five. Errands never reach this screen — water, cutlery, a question and "called the waiter" are the floor's job. A guest unhappy *now* and a guest let down *last time* are the same job, so they share a section.

| Section | The rule |
|---|---|
| Put it right | A concern raised from their phone in the last few minutes, or a guest whose last visit went badly. Either way it is yours to fix. |
| Meet and greet | Top spenders, regulars, and guests who have not been in for a while. A hello from the manager lands differently. |
| Nobody has picked these up | Past its limit with no one closing it. Push it to the section, or take it on yourself. |

Each table is one dense row — who, why, and at most **two** decisions, on one line. There is a single money control, **On the house**, and it opens a sheet that leads with what Pulse suggests and then lets the manager pick anything. Three differently-worded comp buttons on a row was three ways to say the same thing.

---

## Merged

| Was | Now | Why |
|---|---|---|
| Phase 3 "See live" and Phase 4 "Deepen" | One Phase 3 · Anticipate | Turning them on separately gave no distinct value |
| "Welcome them back by name" + "Make them feel remembered" | One greeting whose reason adapts | Same act, different reason |
| "Lead with their usual" + "Recommend the {dish}" | One suggestion | The closest dish to their taste, or their usual. Never both |

## Removed from the plan

| Item | Why |
|---|---|
| Last visit details | Not important right now |
| Multi-guest handling | Covered by the party avatars in Phase 1 |
| Reorder and recommendations split by category | One combined strip is enough |
| QR feedback sentiment as its own pill | Folded into the guest summary |
| "On my way" on a request | A waiter walking to a table does not stop to tell the tablet |
| Crown badge on the floor card | Competed with the action count. One indicator per card |
| Gold "just seated" card treatment | Made a table with nothing to do look louder than one with three things to do |
| "+2" beside the top action | The count badge already says how many |
| "Walk-in" as a table name | A table with no profile is a Guest |
| Per-action icons on unattended capsules | Seven glyphs nobody was taught |
| "The plates are sitting" / "glasses are empty" | Order data, not service data |
| Toast-based undo on a checklist row | Undo sits in the row, where the thumb already is |
| Green "done" rows stacking under the open ones | They refilled the space you just cleared |
| Errands in the Manager tab | Water, cutlery and questions are the floor’s job |
| "Open table" as a manager action | Navigation is not a decision |
| The checklist at Phase 1 | Phase 1 is display only. An empty checklist implied nothing needed doing |
| "Clear and turn the table" | A settled table is already green with a green bill |
| One shared 15-minute limit for everything | Fifteen for cutlery is defensible; fifteen for an unhappy guest is not |
| "Prioritise this table" / "Check in" / "Settle them in" | Instructions to think, not to act. Now: check on them twice as often · go and take their order · pour water and hand them menus |
| "Waited too long" as a label | Overdue says it in one word |
| Five manager sections | Three. A concern now and a bad last visit are the same job |
| Three comp buttons on a manager row | One "On the house" control. The row was mostly empty space and the buttons all said "give something away" |
| Manager cards 150px tall | Dense rows at 62px. A manager reads this standing between tables |

✅ live in the prototype  ◻️ specified, not built
