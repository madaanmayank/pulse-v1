# Pulse — Data Requirements

Every field the prototype actually reads, grouped by entity. Each row shows: **Field — type — source system — used for (which surface)**.

Three source buckets to think in:
- **POS / QR** — already exist in the restaurant (orders, items, tables, servers).
- **Reservations** — booking system (OpenTable / SevenRooms / phone-in log).
- **CRM / Network** — guest profile we build over time (history, preferences, ratings).
- **Derived** — computed by us; not raw input.

> Phase 1 ships with **POS + CRM (own restaurant)** only. Phase 2 adds **Reservations + Network CRM**.

---

## 1 · Guest Profile  *(CRM)*

The "10-second briefing" reads from here. One row per guest, identified by phone.

| Field | Type | Source | Used for |
|---|---|---|---|
| `id` | string | system | join key |
| `name` | string | reservation / waiter intake | briefing title, party header |
| `initials` | string (2 chars) | derived from name | avatars |
| `phone` | E.164 string | reservation / seat-a-guest | recognition / search |
| `visits` | int | derived — count of past orders at this restaurant | "Returning · N visits" |
| `networkVisits` | int | derived — across the multi-restaurant network (Phase 2) | "Network regular" tier |
| `lastVisit` | relative string (`"12 days ago"`) | derived | briefing meta |
| `spend` | int 1–5 (`$`–`$$$$$`) | derived — avg cheque tier | spend dollar signs |
| `avgPerCover` | int (currency) | derived — sum(spend) / covers | "avg $2,400" line |
| `favFood` | string[] | derived — top items ordered | "Usually orders…" bullet, plan engine |
| `favDrinks` | string[] | derived — top drinks ordered | drinks suggestions, wine logic |
| `suggestedDrink` | `{name, type, reason}` | recommendation engine | "Suggest next · Drinks" lead card |
| `suggestedStarter` | `{name, reason}` | recommendation engine | "Suggest next · Starters" |
| `suggestedMain` | `{name, reason}` | recommendation engine | "Suggest next · Main course" |
| `suggestedDessert` | `{name, reason}` | recommendation engine | "Suggest next · Dessert" |
| `upsell` | `{name, reason}` | recommendation engine | upsell row in plan |
| `allergies` | string[] (`"Nuts"`, `"Gluten"`, …) | reservation / waiter intake | red banner in AI summary, kitchen-confirm chip |
| `dislikes` | string[] (`"Spicy food"`) | waiter notes / repeated returns | amber "Avoid…" bullet |
| `occasion` | string (`"Anniversary tonight"`) | reservation / WhatsApp confirm | gold banner, dessert nudge |
| `pastRatings` | int[] 1–5 | post-meal NPS / Google review map | star rating, churn signal |
| `avgRating` | float | derived — mean of `pastRatings` | star display |
| `notes` | string (free text) | waiter / manager | inline cue ("keep courses moving") |
| `vipTier` | enum: `New` / `Regular` / `VIP` / `Network Regular` | derived from spend + visits | VIP star glyph, briefing badge |
| `networkTier` | enum: `silver` / `gold` / `platinum` | network CRM (Phase 2) | network badge |
| `upsellAffinity` | float 0–1 | derived — accept rate on upsell suggestions | optional: confidence shown to waiter |
| `advocacy` | int (Net Promoter style) | derived — refer/return signal | optional: "high advocacy" cue |
| `churnRisk` | bool | derived — declining ratings + days-since-last | red "Churn risk · comp digestif" cue |

**Allergy vocabulary** (controlled): `Nuts, Shellfish, Dairy, Gluten, Vegetarian, Vegan, Halal`.
**Occasion vocabulary** (controlled): `Birthday, Anniversary, Engagement, Celebration, Business, Date`.

---

## 2 · Table (live state)  *(POS + QR)*

One row per physical table. Status and orders update live from the POS.

| Field | Type | Source | Used for |
|---|---|---|---|
| `id` | int | POS / floor plan config | tile number, T-references everywhere |
| `zone` | enum: `indoor` / `patio` / `outdoor` | floor plan config | zone tabs & map rooms |
| `cap` | int (max seats) | floor plan config | capacity warnings, "Seats N" labels |
| `status` | enum: `empty` / `seated` / `ordered` / `attention` / `overdue` / `paid` | POS + derived | tile color, glow, pulses |
| `server` | string (server name) | POS table assignment | "Server James" line |
| `party` | string[] of `guest.id` | seat-a-guest flow | who's at the table |
| `seatedPax` | int | seat-a-guest input | covers count when only one phone is recognised |
| `seatedMin` | int (minutes) | derived — now − seated_at | "seated 38m ago" |
| `course` | enum: `none` / `apps` / `mains` / `dessert` / `check` | POS / derived | current course chip |
| `progress` | bool[4] — [starters, mains, dessert, check] | POS items.served | journey bar, course pips |
| `drinkProgress` | bool[3] — [aperitif, wine, digestif] | POS drinks.served | drink lane |
| `items[]` | array of `{name, course, qty, served}` | POS order lines (food) | "On the table" list, plan engine |
| `drinks[]` | array of `{name, course, qty, served, bottle}` | POS order lines (drinks) | drinks lane, refill logic |
| `drinkAgeMin` | int (minutes since last round poured) | derived | "refill" suggestions |
| `guestRef` / `guestName` | string / string | from seat-a-guest, when no profile exists yet | walk-in display |
| `allergies` | string[] | from seat-a-guest input | carry to table when no profile |
| `occasion` | string | from seat-a-guest input | carry to table when no profile |
| `note` | string | seat-a-guest "additional info" | inline meta |
| `glyphs[]` / `glyphCat[]` | string[] / string[] (parallel arrays) | derived | tile icons (allergy / occasion / VIP / attention) |
| `mx` / `my` | int px, optional | map drag-and-drop | spatial floor map position |

**Status state machine** (server should drive this, not us):

```
empty → seated → ordered → (attention | overdue) → paid → empty
                       ↘                           ↗
                        any course progress moves it back to ordered
```

`attention` and `overdue` are derived (e.g. *seatedMin > expected for current course*).

---

## 3 · Reservation (per booking, per night)  *(Reservations system)*

| Field | Type | Source | Used for |
|---|---|---|---|
| `time` | `HH:MM` | reservation | timeline |
| `name` | string | reservation | row title |
| `phone` | E.164 string | reservation | match to guest profile, seat-a-guest lookup |
| `guestId` | string (nullable) | join on phone to CRM | regular/new badge, prefill |
| `party` | int (pax) | reservation | covers, capacity check |
| `table` | int (nullable) | reservation OR null = assign on arrival | "Table 5" / "No table yet" |
| `zone` | enum | reservation | nice-to-have |
| `status` | enum: `known` / `network` / `new` | derived from `guestId` + network | left-stripe color |
| `allergies` | string[] | reservation form / WhatsApp confirm | red allergy chip |
| `occasion` | string | reservation form | gold occasion chip |
| `notes` | string | reservation / WhatsApp | grey note chip |
| `seated` | bool | derived — set when host confirms seating | "Seated · view T8" CTA |

---

## 4 · Menu *(static config, refresh weekly)*

Used by the suggestion engine to filter for allergy-safe items.

| Field | Type | Source | Used for |
|---|---|---|---|
| `category` | enum: `starters` / `mains` / `desserts` / `drinks` | menu | which lane this fills |
| `name` | string | menu | suggestion display |
| `tags[]` | string[] — `nuts`, `shellfish`, `dairy`, `gluten` | menu | allergy-safe filter (must intersect-empty with guest allergies) |
| `defaultQty` | int (1 or pax) | menu | shared vs. per-person quantity |
| `pairsWith[]` | string[] (optional) | menu | wine/main pairings |

---

## 5 · Restaurant config *(one-time)*

| Field | Type | Used for |
|---|---|---|
| `restaurantId` / `name` / `address` | string | branding strip |
| `floor.rooms[]` | `{zone, w, h, fixtures[]}` | spatial floor map |
| `floor.tables[]` | `{id, zone, cap, x, y, shape}` | drag-positionable tiles |
| `serviceWindows` | `{open, close, doorOpenOffsetMin}` | "doors open in 36 min" |

---

## 6 · Aggregations the views need (read APIs)

These are queries the dashboard makes — they're not new fields, just convenient endpoints.

| Aggregate | Inputs | Returned shape | Used for |
|---|---|---|---|
| `GET /tonight/summary` | date | `{tablesSeated, covers, attention, free, doorOpensInMin}` | floor header stats |
| `GET /tonight/reservations` | date | `Reservation[]` | reservations list |
| `GET /tonight/at-a-glance` | date | `{regulars[], occasions[], allergies[], firstTimers[]}` | right-rail panel |
| `GET /tables/live` | restaurantId | `Table[]` (with `party`, `items`, `drinks`, `progress`) | floor grid + map |
| `GET /guests/lookup?phone=`<br>`GET /guests/lookup?name=` | phone or name | best-match `Guest` + matching `Reservation` (tonight) | seat-a-guest recognition |
| `GET /guests/:id/briefing` | id | full `Guest` + `cues[]` | guest modal |
| `GET /tables/:id/plan` | id | `{cards[], balance[], stage, pax}` from rec engine | "Suggest next" column |

---

## 7 · Write events (what the app emits back)

Anything the host/waiter does on Pulse — capture as events for analytics + to write back into CRM:

| Event | Payload | Notes |
|---|---|---|
| `guest.recognized` | `{guestId, source: 'phone'\|'name'\|'reservation'}` | builds recognition-rate metric |
| `guest.captured` | `{phone?, name?, allergies[], occasion?, notes?}` | new guest from walk-in |
| `table.seated` | `{tableId, party[], pax, source: 'reservation'\|'walk-in'}` | service start time |
| `table.moved` | `{fromId, toId}` | floor moves |
| `table.merged` | `{ids[]}` | big-party setup |
| `course.served` | `{tableId, course}` | journey progression |
| `suggestion.actioned` | `{tableId, course, name, action: 'accepted'\|'shuffled'\|'ignored'}` | feeds `upsellAffinity`, model training |
| `reservation.created` | full reservation | host-side bookings |
| `reservation.seated` | `{reservationId, tableId}` | match-rate to booked-table |
| `ai.summary.viewed` | `{tableId, msShown}` | did the brief actually catch the eye? |
| `ai.search.asked` | `{question}` | only Phase 2+ |

---

## 8 · Suggestion engine (what we'd ask the data team to compute)

The engine reads order history + menu and outputs **per-guest** suggestions. Inputs it needs:

- All past `order_lines` for this guest across visits — item, course, qty, rating-on-that-visit.
- Item co-occurrence (e.g. *80% of lamb orders add truffle fries* → `upsell.reason`).
- Items they've returned multiple times → `usual` tag.
- Allergy-safe set per menu × per guest (precomputed).
- Drinks pacing — average minutes between rounds → triggers refill suggestions.

Three confidence levels that should be returned with each suggestion:
- `usual` — ordered ≥ 50% of visits
- `try` — never ordered, but matches their flavor profile
- `safe` — allergy-screened fallback

---

## What we still need from your side

1. **Phone-as-identity**: do all booking channels capture a phone? If not, what's the secondary key — email, loyalty id?
2. **Ratings ingestion**: where are post-meal ratings collected today (Google, in-app, WhatsApp, none)? `pastRatings` is the single biggest signal for churn risk and tier — if it's missing, we need a path to start collecting it.
3. **Network CRM scope**: which restaurants are "in the network" for Phase 2? That's what powers `networkVisits` and `Priya Rajan`-style cross-restaurant recognition.
4. **POS webhook latency**: target < 5s from "kitchen marks served" → tile updates, or the journey bar lies.
5. **Menu tagging**: someone needs to maintain `tags[]` on the menu. Without allergy tags, the safety guarantee in the AI summary becomes manual.

---

*If it helps the data team, the prototype's JSON shapes (`guests`, `tables`, `reservations`, `MENU`) live in `index.html` near lines 2800–3640 — they're the ground truth for field names.*
