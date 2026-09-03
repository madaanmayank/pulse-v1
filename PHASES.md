# Pulse — what each phase does

Generated from the running registry in `index.html` — every table below is read out of
`PHASE_META`, `SIGNALS`, `CALL_TYPES`, `MG_SECTIONS`, `LEVELS` and `CFG_DEF`, so this file cannot
drift from the app. The same content is on the **Design guide** screen inside the product.

Two rules hold everywhere. **Every action names a physical act** a waiter can perform — never an
instruction to think or to pay closer attention. And **Pulse never says what the data cannot
support**: the card says *last ordered 34 min ago*, not *the plates are sitting*.

Every number in this document is a shipped **default**. All of them are the merchant's to change
in **Settings → Setup** — see [What a merchant can change](#what-a-merchant-can-change).

---

## The three levels

Each answers a different question, so a waiter never has to weigh two of them against each other.

| level | colour | the question | means |
|---|---|---|---|
| `red` | red | Is something wrong? | A guest is waiting, unhappy, or nothing is moving. Go now. |
| `rec` | teal | Is there something to offer? | An offer that lifts the visit or the bill. Never a failure if it is not taken. |
| `gold` | gold | Who is at this table? | Recognition. Who they are, not what they ordered. |

**Amber is not a fourth level.** It means one thing: nobody has closed this. It rings whatever
level was already there — a red concern never turns amber because it aged.

Counts as shipped: 7 red, 8 teal, 4 gold.

---

## Phase 1 · Know — display only

Pulse shows what it already knows about the guest and asks for nothing. No action is raised at
this phase; the value is recognition at the table.

Guest name, masked phone, birthday · new vs returning, visit count, days since last visit · usual
spend · network score and bucket chip · AI guest summary from history, ratings and comments ·
known favourites and reorder · 2–3 recommendations · top taste tags. Plus the table's own live
figures: covers, dishes, tickets, bill, minutes seated. All derived, never typed by staff.

## Phase 2 · Act — the same information as a checklist

6 actions.

| id | what it is | raised when | chased after | whose job |
|---|---|---|---|---|
| `call` | A guest request | The moment they press it | per kind | Whoever is closest |
| `long-stay` | On the table a long time | On the table 180 min | 25 min | Waiter |
| `no-order` | Seated and has not ordered | Seated 1 min with nothing ordered | 10 min | Waiter |
| `recovery` | Last visit went wrong | At once, then every 12 min until they leave | 15 min | Waiter |
| `suggest` | A taste we know about | A taste or a usual on file, nothing ordered | 15 min | Waiter |
| `greet` | A returning guest | From visit 2 | never | Waiter |

## Phase 3 · Anticipate — the guest's phone and the manager

13 more actions, plus the request kinds and every manager touchpoint.

| id | what it is | raised when | chased after | whose job |
|---|---|---|---|---|
| `cart-unsent` | Cart built, order never placed | Items in the cart for 3 min, never sent | 15 min | Waiter |
| `concern-mgr` | A concern, for the manager | The moment a concern is raised | 15 min | Manager |
| `mgr-recovery` | High-value guest let down last time | A let-down guest averaging 60+ | 15 min | Manager |
| `browsing` | Lingering in one part of the menu | 3 min inside one part of the menu | 15 min | Waiter |
| `course-gap` | A long gap since anything was ordered | 25 min since anything was ordered | 15 min | Waiter |
| `drink-refill` | A long gap since a drink was ordered | 20 min since a drink was ordered | 15 min | Waiter |
| `menu-stall` | Long in the menu, nothing added | 6 min in the menu, nothing added | 15 min | Waiter |
| `promo` | A promotion they qualify for | A live promotion applies, nothing ordered | 15 min | Waiter |
| `repeat-item` | One dish opened again and again | One dish opened 3×, never added | 15 min | Waiter |
| `voucher` | A next-visit voucher | The CRM has one they qualify for | 15 min | Waiter |
| `mgr-occasion` | An occasion, for the manager to mark | An occasion on the booking or the profile | never | Manager |
| `occasion` | An occasion | An occasion on the booking or the profile | never | Waiter |
| `vip-welcome` | A top spender | The guest resolves to high value | never | Manager |

---

## What the guest can send

Five buttons on their phone and no free-text field — so Pulse never describes a problem it cannot
see. Where it does not know, it says so: a concern reads *“They could not say what — go and ask.”*

| kind | label | the action | arrives | answer within |
|---|---|---|---|---|
| `concern` | Raised a concern | Hear them out | Phase 3 | 5 min |
| `query` | Has a question | Answer their question | Phase 3 | 5 min |
| `waiter` | Called the waiter | Go to the table | Phase 2 | 5 min |
| `water` | Asked for water | Take water over | Phase 3 | 5 min |
| `cutlery` | Asked for cutlery | Take cutlery over | Phase 3 | 5 min |

At **Phase 2 the guest has one button** and Pulse knows only that it was pressed; the kinds arrive
with Phase 3. Several requests from one table become one capsule, one trip, one tick — the wait
counts from the first ask and the set takes the **tightest** answer time in it. The basic request
cannot be switched off, so a table that pressed a kind the merchant has since disabled still
degrades to *go to the table* rather than vanishing.

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
- **Between checks the card reads `Checked 2× · next in 7m`**, so the table looks held rather than
  forgotten.
- **The manager can see the floor:** *Floor has checked 2×, last 5m ago* — or *Nobody has been
  back yet*. At Phase 3 a high-value guest who was let down also appears in *Put it right* with a
  comp already suggested.

## What happens to an action

| state | means | shown as |
|---|---|---|
| New | raised after the waiter had already looked at this table | ring on the card badge, flag on the row |
| Open | being worked | plain row, badge counts it |
| Overdue | past its own limit with nobody closing it | amber age chip on the card, an Overdue group in the floor row, a section in the Manager tab |
| Done | ticked, right-hand side of the row | strikes through and holds 3 seconds with Undo in place |
| Dismissed | dismissed, left-hand side of the row | same grace period, then gone and not raised again |

A gold action never goes overdue — a greeting cannot be late.

## What a merchant can change

Every number Pulse acts on ships with a considered default and is a merchant's to change in
**Settings → Setup**, one screen, no engineering. Nothing in the engine carries a hard-coded
number, so what Setup says is always what the floor sees. What cannot be changed is the *shape*:
three levels, two limits, five rows.

**Per action** — a switch, and at most two numbers, always the same two:

- **Raise** — when Pulse puts it on the card.
- **Chase** — when it counts as nobody having picked it up. An action with no chase of its own
  inherits the floor rule; a gold one never chases.

Turn an action off and it stops existing: not on a card, not in the floor row, not in the Manager
tab.

**Per request kind** — each of the five buttons gets its own answer time, so a concern can be
tighter than a request for cutlery. Every kind except the basic request can also be switched off.

**Floor rules** — not about any one action, but about how much a person can hold at once:

| rule | default | what it decides |
|---|---|---|
| Chase anything Pulse raised | 15 min | The default limit. Past it, an action is called out by name. |
| Actions shown on one table | 5 rows | Beyond this the rest sit behind one tap. |
| Tables in the floor row | 4 | After this a group collapses to *+N more*. |
| The just-seated window | 10 min | How long a new table gets its own group and softer wording. |
| A new order stays news | 6 min | How long the table stays marked, and the gap actions stay quiet. |
| Undo window | 3 sec | How long a ticked action stays on screen with Undo in place. |

A changed number shows the default it left, one button restores the lot, Settings names how far
from standard the room has drifted before you open Setup at all, and the whole thing persists in
the browser.

## The manager's tab

Three sections, in this order, holding only cases a manager can act on.

1. **Put it right** — A concern raised from their phone in the last few minutes, or a guest whose last visit went badly. Either way it is yours to fix.
2. **Meet and greet** — Top spenders, regulars, and guests who have not been in for a while. A hello from the manager lands differently.
3. **Nobody has picked these up** — Past its limit with no one closing it. Push it to the section, or take it on yourself.

A question or a called waiter never reaches the manager; that is floor work. A concern, a
high-value guest who was let down, and anything nobody has picked up do. Each row carries at most
two decisions plus one *On the house* control, badged with the number of comps Pulse would suggest
for that guest.

## Occasions

A birthday, anniversary, engagement, celebration or business dinner puts a specific gift in the
manager's hands — a dessert with a candle, two glasses of champagne, coffee and petits fours —
plus *Offer 20% off their next visit*, always available. The manager can comp anything manually
as well.

## When an order arrives

Any table sending an order raises a toast naming the table, the item count and the value; the
card's figures move and the table reads as *just ordered*, which suppresses the gap actions.

## Built for landscape tablet

The primary device is a tablet held landscape. Only the chrome was tightened for it — topbar, nav
and controls give back vertical space. The cards themselves are untouched at every size.

---

## Removed from the plan

| dropped | why |
|---|---|
| Amber as a severity | It covered "a task to do", which is two different things: something is wrong, and something is worth offering. A colour that covers two ideas covers neither. Amber now means one thing — nobody has closed this. |
| *"Check on them twice as often"* | A stance for the whole visit dressed up as a checkbox. It could not be performed, so ticking it meant nothing — and once ticked, the table stopped being flagged at all. Replaced by a persistent mark plus a recurring check. |
| Hard-coded thresholds | Every one of them was a guess about somebody else's dining room. They are defaults now, and the merchant owns them. |
| Free-text guest descriptions | The guest's phone has five buttons and no text field, so Pulse cannot know what is wrong. It says so instead of guessing. |
| *"The plates are sitting"* / *"the meal has staled"* | Only order data exists. |
| A second clock for concerns | One limit per kind, set once in Setup, instead of eight rules nobody could recall. |
| Focus / easy-to-consume layout toggle | A second layout to maintain for the same screen. The one layout was simplified instead. |
| A crown *and* a gold card for the same guest | One indicator per card. |
| The sentiment chip | Nothing in the data supports it. |
| Clear-and-close table flow | Not needed for the story this prototype tells. |
| Phases 3 and 4 as separate steps | Merged — anticipation and the manager's tools ship together. |
