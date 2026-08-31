# Pulse — Actionable Hospitality: the use-case catalogue

**From guest intelligence to actionable hospitality.** Pulse already knows who the guest is. This phase answers the question that follows: *what matters about this table right now, why does it matter, and what should we do about it?*

Every use case below follows one shape:

> **Why this matters → What to do → Done**

**Legend** — **●** live in the prototype · **○** buildable on data we already hold · **◌** needs a new signal we don't yet receive

**Where we are:** 31 use cases are live and working in the prototype. The full catalogue below runs to ~130, of which ~85 are buildable on data we hold today. That gap is the roadmap, not a gap in the thinking.

---

## The one rule that shapes everything

A signal earns its place on the table screen only if it changes what a staff member does. If it doesn't, it belongs deeper in the guest profile. We also never invent a signal we cannot actually measure — there is no "the main course is late" alert, because we do not yet receive kitchen timing. Nothing on screen claims knowledge we don't have.

---

## A · Safety & dietary
*Red lives here and almost nowhere else.*

| | Use case | The action we recommend | Who |
|---|---|---|---|
| **●** | An ordered plate conflicts with a declared allergy at the table | Stop the plate and check with the kitchen | Waiter |
| **●** | Kitchen not yet briefed on this table's restriction | Brief the kitchen on this table | Waiter |
| ○ | Allergy captured at the door but never saved to the profile | Save it to the guest profile | Waiter |
| ○ | A declared allergy can't be verified against our menu tags | Verify by hand with the kitchen | Waiter |
| ◌ | Their unsent basket contains a dish that clashes with their allergy | Stop them before they send it | Waiter |
| ◌ | Guest is reading allergen information before ordering | Offer to talk them through it | Waiter |
| ◌ | Allergen info viewed but no dietary flag on file | Ask, then save it | Waiter |

The allergy itself always shows as a permanent red context chip — unmissable reference, separate from the action.

## B · Service opportunities — the next best action
*Turning what we know into a small, specific piece of hospitality.*

| | Use case | The action we recommend | Who |
|---|---|---|---|
| **●** | Occasion known for tonight (birthday, anniversary, engagement) | Acknowledge the occasion | Waiter |
| **●** | Occasion still unmarked as the meal reaches dessert or the check | Mark it before they leave | Waiter |
| **●** | Returning guest, three or more visits | Welcome them back by name | Waiter |
| **●** | Known across the group but first time at this restaurant | Introduce the room properly | Waiter |
| **●** | First visit in a long while | Make them feel remembered | Waiter |
| **●** | They have a usual and haven't ordered yet | Lead with their usual | Waiter |
| **●** | Something they add to this order most visits | Mention the specific item | Waiter |
| ○ | Their usual drink, before they ask | Offer it by name | Waiter |
| ○ | A pairing that fits the main they just ordered | Suggest the pairing | Waiter |
| ○ | Dessert they always finish with | Have it ready | Waiter |
| ○ | Preference relevant to what they're choosing right now | Point them at the right options | Waiter |
| ○ | Guest who rated a specific dish highly last time | Mention it's on tonight | Waiter |
| ◌ | Party where the group's tastes conflict | Suggest a middle path | Waiter |

## C · Live guest behaviour — what the floor cannot see
*Our real differentiator: the guest is on our QR while they sit at the table, so we can see intent before it becomes a complaint.*

| | Use case | The action we recommend | Who |
|---|---|---|---|
| **●** | Explicit request raised from the table | Respond to the table | Waiter |
| **●** | Extended browsing of a menu section without ordering *(flagship)* | Offer help choosing | Waiter |
| **●** | Menu open for minutes with nothing viewed at all | Offer to walk them through the menu | Waiter |
| **●** | Same dish opened repeatedly without being added | Talk them through that dish | Waiter |
| ○ | Browsed, then went quiet with nothing ordered | Check in before they disengage | Waiter |
| ○ | Occupied table with no QR session at all | Offer a paper menu or take it verbally | Waiter |
| ○ | Dessert section opened after mains | Close the dessert conversation now | Waiter |
| ◌ | Order built but never submitted | Ask if they meant to send it | Waiter |
| ◌ | Strong interest in an item, then removed from basket | Ask if they need help deciding | Waiter |
| ◌ | Repeated returns to a premium item | Upsell readiness — mention it | Waiter |
| ◌ | Large party where only one phone is ordering | Offer to take the rest verbally | Waiter |
| ◌ | Taking far longer than the venue norm to decide | Offer ordering assistance | Waiter |
| ◌ | Guest requested the bill from their phone | Bring the check | Waiter |
| ◌ | Order failed to submit — nothing reached the kitchen | Take the order again immediately | Waiter |

## D · Experience attention — live and measurable
*Only signals we can genuinely calculate. Each one is explainable in a single sentence.*

| | Use case | The action we recommend | Who |
|---|---|---|---|
| **●** | Table has been waiting past its stage | Check the pass and update them | Waiter |
| **●** | Seated a while with no order placed | Check in with the table | Waiter |
| **●** | Table flagged as needing attention | Check in with the table | Waiter |
| **●** | Round poured long ago, glasses running low | Offer another round | Waiter |
| **●** | Bill paid, table not yet cleared | Clear and turn the table | Waiter |
| ○ | Ordering activity started then stopped | Check in with the table | Waiter |
| ○ | More attention items on one table than we can show | Escalate — this table is compounding | Waiter |
| ◌ | Long gap since the last course was served | Chase the kitchen | Waiter |
| ◌ | Table waiting on the check | Bring it now | Waiter |
| ◌ | Large party progressing unevenly | Re-sync the courses | Waiter |
| ◌ | Nobody has touched the table in N minutes | Make a pass | Waiter |

## E · Relationship, recognition & service recovery
*Recovery is historical risk, not a live event — so it looks visually different, and it starts at the door.*

| | Use case | The action we recommend | Who |
|---|---|---|---|
| **●** | Guest had a specific service failure on a previous visit | Prioritise this table | Waiter |
| **●** | Ratings have been slipping across recent visits | Check in early and often | Waiter |
| **●** | Arriving with a previous issue *(shown on the reservation, pre-arrival)* | Give them your strongest table and server | Host |
| ○ | Guest returning after a complaint was resolved | Confirm it's right this time | Waiter |
| ○ | High-value guest whose average rating is drifting down | Extra care tonight | Waiter |
| ○ | Recovery attempted but never followed through | Close the loop | Manager |
| ◌ | The same failure is happening again on this visit | Escalate immediately | Manager |
| ◌ | Last chance to change the rating before the feedback prompt | Intervene now | Manager |

## F · Managerial intelligence — where the next ten minutes go
*A manager is not another waiter. They can't visit every table, so we surface only where their presence changes the outcome.*

| | Use case | The action we recommend | Who |
|---|---|---|---|
| **●** | Recovery on a loyal or high-value guest | Visit the table personally | Manager |
| **●** | Flagged for attention and nobody has even opened the table | Get someone to the table | Manager |
| **●** | Several attention signals converging on one table | Visit the table | Manager |
| **●** | Occupied table with no server assigned | Assign a server | Manager |
| **●** | One section is underwater | Rebalance the section | Manager |
| **●** | At-risk regular is dining right now | Win them back tonight | Manager |
| **●** | High-value table has paid — farewell window | See them out yourself | Manager |
| **●** | Priority guest currently dining | Personally welcome them | Manager |
| **●** | Guest who introduces other guests | Introduce the chef | Manager |
| **●** | Occasion for one of your best guests | Send something to the table | Manager |
| **●** | Large party on the floor | Check it yourself once | Manager |
| ○ | VIP on their very first visit here | Set the tone personally | Manager |
| ○ | Overdue service on a high-value table | Intervene before it's remembered | Manager |
| ○ | Urgent action seen but not actioned past a threshold | Take it over | Manager |
| ◌ | Explicit guest request left unanswered past standard | Respond yourself | Manager |

## G · The action & learning layer
*Not a category of intelligence — a capability that runs across all of them.*

| | Capability | State |
|---|---|---|
| **●** | Mark any recommended action done in one tap | Live |
| **●** | Completed actions collapse to a quiet confirmation, clearing the screen | Live |
| **●** | Undo a completion | Live |
| **●** | Every completion is recorded: signal → recommended action → who acted | Live |
| **●** | "Remember this" — staff teach the system in two taps and one line | Live |
| **●** | Saved memory surfaces as table context on the next visit | Live |
| ○ | Time-to-action per signal type | Roadmap |
| ○ | Which recommendations staff consistently ignore (and should be retired) | Roadmap |
| ◌ | Did the intervention actually improve the guest's rating? | Roadmap |

---

## How the interface stays glanceable

The catalogue is large. The screen is not. That's deliberate:

- **Never more than two action cards per table.** Everything else collapses behind "+N more" or drops to context. Nine simultaneous insights would recreate the problem we're solving.
- **Priority order is fixed:** something needs attention now → something can materially improve the experience → useful context.
- **The action is the dominant visual element**, not the explanation. One tap, from the exact point of touch.
- **Colour is rationed.** Red is only for immediate intervention — across a full floor of twelve occupied tables, four red cards. Amber for attention. Neutral for opportunity. A sea of red badges means nothing is important.
- **Progressive disclosure.** First glance: what do I need to know. Second: what do I need to do. Only if asked: tell me more about the guest.
- **The floor is the dashboard.** Each tile carries one primary indicator — a health dot, an action count, and the single top action. No icon piles.
- **No AI-branded interface.** Copy is operational and human: "Guest may need assistance", never "AI detected a potential assistance opportunity". Staff don't care what generated it.

## Experience health

Every occupied table resolves to green, amber or red, and the reason is always a sentence a human wrote:

| Level | Means | Example reason |
|---|---|---|
| 🔴 Red | Immediate intervention | "Truffle Arancini conflicts with this table" |
| 🟡 Amber | Needs attention soon | "Ordering hasn't started" |
| 🟢 Green | Nothing needs you | "Experience looks healthy" |

Health is never an opaque score. If we can't explain it in one line, we don't show it.

## What we are deliberately not building yet

- **Another CRM.** We're not trying to expose every field we hold.
- **Another analytics dashboard.** No waiter should have to interpret a chart mid-service.
- **A chatbot for staff.** The waiter shouldn't have to ask. Pulse surfaces what matters proactively.
- **Deep prediction.** Abandonment prediction, refill forecasting and network-level personalisation all come *after* we've proven the basic loop: identify a moment, recommend an action, and have staff actually take it.

## What the next phase has to prove

> Can Pulse reliably identify meaningful moments during a live service and turn them into actions that staff actually take?

Everything else follows from that. The action log exists precisely so we can answer it with data rather than opinion.

---

*Full engineering specification — triggers, ranking maths, field definitions — is in `SIGNAL_ENGINE_SPEC.md`. Data requirements are in `DATA_REQUIREMENTS.md`. Product positioning is in `PRODUCT_BRIEF.md`. Live prototype: https://madaanmayank.github.io/pulse-v1/*
