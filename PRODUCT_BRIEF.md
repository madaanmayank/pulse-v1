# Pulse — Product Brief

*Paste this at the top of any conversation to give the model the full picture of what we're building.*

---

## What Pulse is, in one line

**Pulse is a guest-intelligence tablet app for the world's top premium dine-in restaurants** — a "10-second briefing" a waiter glances at during service, not a system they operate.

## The problem we're solving

Fine-dining service happens in two places at once: at the table (warmly, invisibly) and in the waiter's head (a growing pile of things they must remember — this guest's allergy, that couple's anniversary, the regular who always orders the Barolo, the party of six who's mid-mains and now needs dessert cued). The best waiters carry it; most don't. Guests can tell the difference within 60 seconds of sitting down, and it decides whether they come back.

Existing tools are the opposite of what a floor needs: POS terminals designed for order entry, reservation systems designed for the phone-facing host, dashboards designed for the manager's laptop. **Nothing helps the person actually walking the room.**

## The core insight

**A waiter should never read the app. They should glance at it.**

Every design decision follows from that. If a screen requires reading a paragraph, we've failed. If a table's status can't be understood in half a second from across the room, we've failed. The AI's job is to *pre-chew* the information — into a single sentence, a colored border, one pulsing tile — so the human can stay warm and attentive at the table.

## Who's the user

- **Primary**: the floor waiter / captain during a live service.
- **Secondary**: the host at the door (seating flow, reservations).
- **Not**: the manager (they get their own view later), the kitchen (POS handles that), the guest (they never see Pulse — they see their waiter being magically prepared).

## The two phases we've built

The product ships in phases because the data does. Phase 1 works with what a restaurant already has; Phase 2 unlocks when they wire in reservations and a shared guest network.

**Phase 1 — The Briefing**
Just QR order data + the restaurant's own guest history. Delivers:
- Live floor view (grid + spatial map) where every table wears its state as a glowing border/pulse.
- A one-glance "Pulse AI service summary" per table — allergies, occasion, pace, plus what to bring next with quantities scaled to the party.
- Seat-a-guest flow (walk-ins), even without reservations.
- Guest briefing card by phone lookup — history, ratings, favorites.

**Phase 2 — The Network**
Adds reservations (with phone recognition), cross-restaurant guest network, AI concierge search ("what wine for Anika's lamb?"), editable spatial floor map, predictive cues (churn risk → comp digestif). This is the default the app now opens to.

## The core loop

**Recognise → Suggest → Serve → Turn.**

1. Guest sits down (walk-in typed in, or reservation seated).
2. Pulse recognizes them by phone or name, plays a fast cinematic ("Loading profile → Reading preferences → Curating their evening") and takes the waiter straight to the table.
3. Table view shows the AI summary + course-scaled "Suggest next" (with allergy safety guaranteed).
4. As the QR order stream lands, the plan rewrites itself. Journey bar advances through Seated → Starters → Mains → Dessert → Check.
5. Bill paid → tile turns a dashed green "Ready to clear & turn."

## The aesthetic

The reference is CaratLane / fine-watch / fine-fashion editorial. It's a tool that lives on a tablet in a $200-cover restaurant, so it must not look like SaaS. Non-negotiables:

- **Warm dark canvas** — near-black with a candlelight glow, film grain, and vignette. Not cold-blue "developer dark."
- **Cormorant Garamond** for every brand/title moment (wordmark, page titles, guest names). **Inter** for UI, **Geist Mono** for numerals.
- **Champagne** (#d8c39a) is the single accent — replaces the old teal. Applied to CTAs, the "Pulse AI" badge (with shimmer animation), stars, focus rings.
- **Motion is choreography, not decoration** — staggered entrances, live count-up numbers, cursor-tracking spotlights on cards, 3D place-card tilt, point-of-touch ripples, a champagne pulse on tables that need attention. All respect `prefers-reduced-motion` and work on touch (timed, not hover-dependent).
- **Glass UI is an optional theme toggle** (frosted translucent panels) for the same content — the app supports both.

## Interaction rules that matter

- Never seat a table silently. Clicking an empty tile always opens the "Seat a guest" form (phone recognition → autofill from CRM or from tonight's reservations).
- The guest section in a table view is entirely tappable (opens the briefing).
- Search reservations by name, table, occasion, or allergy — live filter, input keeps focus.
- Follow-along tutorial: while the tour runs, exactly two things are tappable — the tour card, and (on click steps) the highlighted target. Everything else is swallowed. There's always a "Do it for me" fallback so the user can never get stuck.

## What Pulse is NOT

- Not a POS. It reads order data; it doesn't take orders.
- Not a reservation booking system. It reads bookings; it doesn't handle payments/holds.
- Not a KDS (kitchen display). It shows what's been served, not what's being cooked.
- Not a manager dashboard. That's a separate future surface.
- Not multi-tenant SaaS priced $99/mo. Positioning is the top 100 restaurants globally.

## Data model, at a glance

Four entities. Full spec in `DATA_REQUIREMENTS.md`.

- **Guest** — phone is identity. Visits, ratings, favorites, allergies, occasion, tier, churn flag.
- **Table** — id, zone, capacity, status (empty/seated/ordered/attention/overdue/paid), party (guest ids), live items[], drinks[], progress[4] course flags.
- **Reservation** — time, name, phone, party, table, occasion, allergies, seated bool.
- **Menu** — items with allergen tags for the safety filter.

Suggestion engine outputs three confidence tiers per recommendation: `usual` (ordered ≥50% of visits), `try` (matches profile), `safe` (allergy-screened fallback).

## The signature moments

If you're designing or writing anything new for Pulse, these are the moments that define the product — everything else supports them:

1. **The Pulse AI service summary** at the top of a table view. One paragraph that reads like a captain briefing a waiter, bolding the words that must be acted on.
2. **The assignment cinematic** when a table is opened. ~3 seconds of "Loading profile → Reading preferences → Curating their evening" that redirects the waiter to the assigned table with a champagne pulse.
3. **A tile that needs attention** — soft pulsing amber (attention) or fast red (overdue), scannable from across the room.
4. **The reservations list**, serif guest names in Cormorant, one tap to seat a whole booking with all its context carried forward.

## Prompt-ready one-liner

If you're generating for Pulse, hold this in mind:

> *"A tablet app for a captain at a top-100 restaurant. Warm dark, champagne accents, Cormorant serif. Motion is choreography. Never make the waiter read — pre-chew every insight into one glanceable line, one glowing border, one confident suggestion. The goal is a service where the guest thinks their waiter simply remembers everything."*

---

*Live prototype: https://madaanmayank.github.io/pulse-v1/*
