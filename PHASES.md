# Pulse — what each phase does

Generated from the running registry in `index.html` — every table below is read out of
`PHASE_META`, `SIGNALS`, `CALL_TYPES` and `MG_SECTIONS`, so this file cannot drift from the app.
The same content is on the **Design guide** screen inside the product.

Numbers that govern behaviour: a guest request is chased after **5 min**, anything Pulse raised
itself goes overdue after **15 min**, a table shows at most **5** actions at once, the floor row
shows at most **4** tables before it collapses into *+N more*, and an at-risk table is checked
again every **12 min**.

Two rules hold everywhere. **Every action names a physical act** a waiter can perform — never an
instruction to think or to pay closer attention. And **Pulse never says what the data cannot
support**: the card says *last ordered 34 min ago*, not *the plates are sitting*.

---

## Phase 1 · Know — display only

Pulse shows what it already knows about the guest and asks for nothing. No action is raised at
this phase; the value is recognition at the table.

Guest name, visit count, average spend, score and bucket chip, tastes, allergies, the last
visit's date, and the table's own live figures — covers, dishes, tickets, bill, minutes seated.
All derived, never typed by staff.

## Phase 2 · Act — the same information as a checklist

6 actions.

| id | what it is | level | whose job |
|---|---|---|---|
| `call` | A guest request | Intervene now | Whoever is closest |
| `recovery` | Last visit went wrong | Intervene now | Waiter |
| `long-stay` | On the table a long time | A task to do | Waiter |
| `no-order` | Seated and has not ordered | A task to do | Waiter |
| `greet` | A returning guest | Something to say | Waiter |
| `suggest` | A taste we know about | Something to say | Waiter |

## Phase 3 · Anticipate — the guest's phone and the manager

13 more actions, plus the request kinds and every manager touchpoint.

| id | what it is | level | whose job |
|---|---|---|---|
| `cart-unsent` | Cart built, order never placed | Intervene now | Waiter |
| `concern-mgr` | A concern, for the manager | Intervene now | Manager |
| `mgr-recovery` | High-value guest let down last time | Intervene now | Manager |
| `browsing` | Lingering in one part of the menu | A task to do | Waiter |
| `course-gap` | A long gap since anything was ordered | A task to do | Waiter |
| `drink-refill` | A long gap since a drink was ordered | A task to do | Waiter |
| `menu-stall` | Long in the menu, nothing added | A task to do | Waiter |
| `repeat-item` | One dish opened again and again | A task to do | Waiter |
| `mgr-occasion` | An occasion, for the manager to mark | Something to say | Manager |
| `occasion` | An occasion | Something to say | Waiter |
| `promo` | A promotion they qualify for | Something to say | Waiter |
| `vip-welcome` | A top spender | Something to say | Manager |
| `voucher` | A next-visit voucher | Something to say | Waiter |

---

## What the guest can send

Five buttons on their phone and no free-text field — so Pulse never describes a problem it
cannot see. Where it does not know, it says so: a concern reads *“They could not say what — go
and ask.”*

| kind | label | the action | arrives |
|---|---|---|---|
| `concern` | Raised a concern | Hear them out | Phase 3 |
| `query` | Has a question | Answer their question | Phase 3 |
| `waiter` | Called the waiter | Go to the table | Phase 2 |
| `water` | Asked for water | Take water over | Phase 3 |
| `cutlery` | Asked for cutlery | Take cutlery over | Phase 3 |

At **Phase 2 the guest has one button** and Pulse knows only that it was pressed; the kinds
arrive with Phase 3. Several requests from one table become one capsule, one trip, one tick —
the wait counts from the first ask and the set takes the tightest limit on it.

## A table you have to keep close

The one state that is not a task. When the guest's last visit went wrong, Pulse does **not** ask
the floor to check on them more often: nobody can do that, and a stance ticked off as a checkbox
stops being true the moment it is ticked.

So **the state stays and the action recurs.**

- **`At risk` sits on the card** from sitting down to paying, and has no tick. It is a fact about
  the table, not a job. Set by one thing only — something specific went wrong last visit; never
  inferred from a score.
- **The check is the action.** *Go and check on them* → *Go back and check on them*. A real trip
  to a real table, so it can be honestly ticked.
- **Ticking it records the visit** instead of closing anything: it logs that someone went and
  re-arms the clock. The action returns every 12 min and flags as **New** each time — even to a
  waiter who has already looked at this table.
- **Between checks the card reads `Checked 2× · next in 7m`**, so the table looks held rather
  than forgotten.
- **The manager can see the floor:** *Floor has checked 2×, last 5m ago* — or *Nobody has been
  back yet*. At Phase 3 a high-value guest who was let down also appears in *Put it right* with a
  comp already suggested.

## What happens to an action

| state | means | shown as |
|---|---|---|
| New | raised after the waiter had already looked at this table | ring on the card badge, flag on the row |
| Open | being worked | plain row, badge counts it |
| Overdue | past its own limit with nobody closing it — 5 min for a guest request, 15 for anything Pulse raised itself | age chip on the card, an Overdue group in the floor row, a section in the Manager tab |
| Done | ticked, right-hand side of the row | strikes through and holds three seconds with Undo in place |
| Dismissed | dismissed, left-hand side of the row | same grace period, then gone and not raised again |

Lateness is an additive ring, never a colour change — a red concern never becomes amber because
it aged.

## The manager's tab

Three sections, in this order, holding only cases a manager can act on.

1. **Put it right** — A concern raised from their phone in the last few minutes, or a guest whose last visit went badly. Either way it is yours to fix.
2. **Meet and greet** — Top spenders, regulars, and guests who have not been in for a while. A hello from the manager lands differently.
3. **Nobody has picked these up** — Past its limit with no one closing it. Push it to the section, or take it on yourself.

A question or a called waiter never reaches the manager; that is floor work. A concern, a
high-value guest who was let down, and anything nobody has picked up do. Each row carries at most
two decisions plus one *On the house* control, badged with the number of comps Pulse would
suggest for that guest.

## Occasions

A birthday, anniversary, engagement, celebration or business dinner puts a specific gift in the
manager's hands — a dessert with a candle, two glasses of champagne, coffee and petits fours —
plus *Offer 20% off their next visit*, always available. The manager can comp anything manually
as well.

## When an order arrives

Any table sending an order raises a toast naming the table, the item count and the value; the
card's figures move and the table reads as *just ordered* for six minutes, which suppresses the
gap-since-last-order actions.

## Built for landscape tablet

The primary device is a tablet held landscape. Only the chrome was tightened for it — topbar, nav
and controls give back vertical space. The cards themselves are untouched at every size.

---

## Removed from the plan

| dropped | why |
|---|---|
| *“Check on them twice as often”* | A stance for the whole visit dressed up as a checkbox. It could not be performed, so ticking it meant nothing — and once ticked, the table stopped being flagged at all. Replaced by a persistent mark plus a recurring check. |
| Free-text guest descriptions | The guest's phone has five buttons and no text field, so Pulse cannot know what is wrong. It says so instead of guessing. |
| *“The plates are sitting”* / *“the meal has staled”* | Only order data exists. |
| A second clock for concerns | One limit for every request. A concern is set apart by whose job it is, not by a second timer. |
| Focus / easy-to-consume layout toggle | A second layout to maintain for the same screen. The one layout was simplified instead. |
| A crown *and* a gold card for the same guest | One indicator per card. |
| The sentiment chip | Nothing in the data supports it. |
| Clear-and-close table flow | Not needed for the story this prototype tells. |
| Phases 3 and 4 as separate steps | Merged — anticipation and the manager's tools ship together. |
