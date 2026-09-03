# Pulse — what each phase does

Generated from the running registry in `index.html` — every table below is read out of
`PHASE_META`, `SIGNALS`, `CALL_TYPES`, `MG_SECTIONS`, `LEVELS`, `SETUP` and `CFG_DEF`, so this file
cannot drift from the app. The same content is on the **Design guide** screen inside the product.

Two rules hold everywhere. **Every action names a physical act** a waiter can perform — never an
instruction to think or to pay closer attention. And **Pulse never says what the data cannot
support**: the card says *last ordered 34 min ago*, not *the plates are sitting*.

Every number here is a shipped **default**, and all of them are the merchant's to change in
**Settings → Setup**.

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

## Two clocks, and only two

| | limit | what happens past it |
|---|---|---|
| **A guest request** | per kind, 5 min as shipped | Pulse names the table; several requests from one table take the tightest of them |
| **Anything Pulse raised itself** | **15 min, shared by every action** | Pulse calls the table out by name on the card, in the strip above the floor, and in the Manager tab |

Every action shares the one chase — there is no per-action chase to remember. A gold action never
chases at all, because a greeting cannot be late.

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

| id | what it is | what it does, and when | its number | whose job |
|---|---|---|---|---|
| `call` | Waiter called | "Go to the table" — the moment they press it. How long the floor has to answer is set per kind, above. | — | Whoever is closest |
| `long-stay` | On the table a long time | "Offer the check" — once they have been on the table this long. | **180** — show it after | Waiter |
| `no-order` | Seated and has not ordered | "Go and take their order" — once they have been seated this long with nothing ordered. | **10** — show it after | Waiter |
| `recovery` | Last visit went wrong | "Go and check on them" — at once, then again this often for as long as they are in. | **20** — go back every | Waiter |
| `suggest` | A taste we know about | "Recommend the Wagyu Ribeye" — whenever a taste or a usual is on file and nothing is ordered. | — | Waiter |
| `greet` | A returning guest | "Greet them as Jessica" — from this visit number onward. | **2** — from visit | Waiter |

## Phase 3 · Anticipate — the guest's phone and the manager

13 more actions, plus the request kinds and every manager touchpoint.

| id | what it is | what it does, and when | its number | whose job |
|---|---|---|---|---|
| `cart-unsent` | Cart built, order never placed | "Ask them to send their order" — once items have sat in their cart this long, unsent. | **3** — show it after | Waiter |
| `concern-mgr` | A concern, for the manager | "Visit the table" — the moment a concern is raised from the table. | — | Manager |
| `mgr-recovery` | High-value guest let down last time | "Visit the table" — for a guest whose last visit fell short, and only above this average spend. | **60** — only above | Manager |
| `browsing` | Lingering in one part of the menu | "Offer to choose a wine with them" — once they have been in one part of the menu this long. | **3** — show it after | Waiter |
| `course-gap` | A long gap since anything was ordered | "Offer the next course" — once nothing at all has been ordered for this long. | **25** — show it after | Waiter |
| `drink-refill` | A long gap since a drink was ordered | "Offer another round" — once no drink has been ordered for this long. | **20** — show it after | Waiter |
| `menu-stall` | Long in the menu, nothing added | "Talk them through the menu" — once the menu has been open this long with nothing added. | **6** — show it after | Waiter |
| `promo` | A promotion they qualify for | "Mention the promotion" — whenever a live promotion applies and nothing is ordered. | — | Waiter |
| `repeat-item` | One dish opened again and again | "Talk them through the Wagyu Ribeye" — once one dish has been opened this many times without being added. | **3** — show it after | Waiter |
| `voucher` | A next-visit voucher | "Offer the voucher" — whenever the CRM has one they qualify for. | — | Waiter |
| `mgr-occasion` | An occasion, for the manager to mark | "Mark the occasion" — whenever an occasion is on the booking or the profile. | — | Manager |
| `occasion` | An occasion | "Mention the anniversary" — whenever an occasion is on the booking or the profile. | — | Waiter |
| `vip-welcome` | A top spender | "Visit the table" — whenever the guest resolves to high value. | — | Manager |

---

## What the guest can send

Five buttons on their phone and no free-text field — so Pulse never describes a problem it cannot
see. Where it does not know, it says so: a concern reads *"They could not say what — go and ask."*

| kind | label | the action | arrives | answer within |
|---|---|---|---|---|
| `concern` | Raised a concern | Hear them out | Phase 3 | 5 min |
| `query` | Has a question | Answer their question | Phase 3 | 5 min |
| `waiter` | Called the waiter | Go to the table | Phase 2 | 5 min |
| `water` | Asked for water | Take water over | Phase 3 | 5 min |
| `cutlery` | Asked for cutlery | Take cutlery over | Phase 3 | 5 min |

At **Phase 2 the guest has one button** and Pulse knows only that it was pressed; the kinds arrive
with Phase 3. Several requests from one table become one capsule, one trip, one tick — the wait
counts from the first ask and the set takes the **tightest** answer time in it. Each kind but the
basic request can also be switched off; a table that pressed a kind since disabled degrades to
*go to the table* rather than vanishing.

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
  re-arms the clock. The action returns every 20 min and flags as **New** each time — even to a
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
| Overdue | 15 min with nobody closing it | amber age chip on the card, an Overdue group in the strip above the floor, a section in the Manager tab |
| Done | ticked, right-hand side of the row | strikes through and holds 2 seconds with Undo in place |
| Dismissed | dismissed, left-hand side of the row | same grace period, then gone and not raised again |

## What a merchant can change

Every number Pulse acts on ships with a considered default and is a merchant's to change in
**Settings → Setup**, one screen, no engineering. Nothing in the engine carries a hard-coded
number, so what Setup says is always what the floor sees. What cannot be changed is the *shape*:
three levels, two clocks, five rows.

**Per action** — a switch, and **one number: how long before it appears.** The sentence beside it
says what that time is measured from, so a stepper never has to be guessed at. Turn an action off
and it stops existing: not on a card, not in the strip above the floor, not in the Manager tab.

**Per request kind** — each of the five buttons gets its own answer time, so a concern can be
tighter than a request for cutlery.

**Floor rules** — how long anything may sit ignored, and how much a person can hold at once:

| rule | default | what it decides |
|---|---|---|
| One chase, for everything Pulse raises | 15 min | Every action above shares this one limit. |
| Actions shown on one table | 5 rows | Open a table and this many actions are listed, most urgent first; anything beyond sits behind one tap. |
| Tables named in the strip above the floor | 4 tables | The strip across the top of the Table screen names every table waiting on staff. |
| The just-seated window | 10 min | A table this new gets its own group in the strip and softer wording, so arriving never reads as a failure. |
| A new order stays news | 5 min | How long a table stays marked after an order lands, and how long the gap-since-last-order actions stay quiet. |

A changed number shows the default it left, one button restores the lot, Settings names how far
from standard the room has drifted before you open Setup at all, and the whole thing persists in
the browser.

Two numbers are deliberately **not** here: the undo window (2 seconds — a feel, not a policy) and
the number of levels.

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
| A chase per action | Nineteen limits is nineteen things to remember and nobody remembers any of them. One shared chase, one number, one place. |
| The undo window as a setting | A feel, not a policy. Fixed at two seconds. |
| Amber as a severity | It covered "a task to do", which is two different things: something is wrong, and something is worth offering. A colour that covers two ideas covers neither. Amber now means one thing — nobody has closed this. |
| *"Check on them twice as often"* | A stance for the whole visit dressed up as a checkbox. It could not be performed, so ticking it meant nothing — and once ticked, the table stopped being flagged at all. Replaced by a persistent mark plus a recurring check. |
| Hard-coded thresholds | Every one of them was a guess about somebody else's dining room. They are defaults now, and the merchant owns them. |
| Free-text guest descriptions | The guest's phone has five buttons and no text field, so Pulse cannot know what is wrong. It says so instead of guessing. |
| *"The plates are sitting"* / *"the meal has staled"* | Only order data exists. |
| A second clock for concerns | One answer time per kind, set once in Setup. |
| Focus / easy-to-consume layout toggle | A second layout to maintain for the same screen. The one layout was simplified instead. |
| A crown *and* a gold card for the same guest | One indicator per card. |
| The sentiment chip | Nothing in the data supports it. |
| Clear-and-close table flow | Not needed for the story this prototype tells. |
| Phases 3 and 4 as separate steps | Merged — anticipation and the manager's tools ship together. |
