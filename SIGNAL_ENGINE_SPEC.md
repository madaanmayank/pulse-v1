# PULSE — ACTION INTELLIGENCE ENGINE · CONSOLIDATED IMPLEMENTATION SPEC v2

**Target file:** `/Users/mayankmadaan/pulse-prototype/index.html` (currently **5495 lines** — CSS `<style>` ~11–2790, HTML body ~2792–2930, JS `<script>` 2931–5493)

**IMPORTANT — the contract in the brief is one revision stale.** A **v1 of this engine already exists in the file** and this spec is an *upgrade*, not a greenfield build. Verified as already implemented:

| Already built | Where |
|---|---|
| `MAX_PRIMARY = 2`, `actionState{}`, `actionLog[]`, `actKey`, `isActioned`, `daysSince` | 3876–3891 |
| `signalContext(t, pol)` — the single context object every trigger reads | 3893–3920 |
| `SIGNALS[]` catalog — 22 signals with `{id,tier,severity,audience,icon,live,historical,score,when,title,why,action,done}` | 3922–4104 |
| `tableSignals(t,pol)` → `{attention, opportunity, manager, done, primary, overflow, health, ctx}` | 4111–4147 |
| `tableHealth(t, attention, c)` → `{level, reason, count}` | 4151–4161 |
| `markAction` / `undoAction` / `renderActionCard` / `renderDoneRow` / `renderTableActions` / `toggleActMore` | 4163–4235 |
| `renderContextStrip`, `renderGuestSection`, `toggleDisclose` | 4237–4297 |
| Remember This: `openRemember` / `pickRemCat` / `saveRemember` + `#remember-modal` (6 categories) | 4300–4336, HTML 2834–2866 |
| Manager view: `renderManager()`, `#view-manager`, `pol.showManager: p>=2` | 5113–5250, 3196 |
| Tile health dot + open-count + action footer in `renderTile` | 3329–3352 |
| CSS: `.sec-head(.urgent)`, `.act-card(.red/.amber/.neutral/.historical/.mgr/.clearing)`, `.act-title`, `.act-why`, `.act-do(.act-do-lbl/.act-do-done)`, `.act-done-row/.act-undo`, `.act-more`, `.act-clear/.ac-ic`, `.health-pill/.hdot`, `.tile-health/.hdot/.hcount`, `.tile-action`, `.ctx-strip/.ctx-chip`, `.disclose/.disclose-body`, `.kv-grid/.kv`, `.rem-btn/.rem-cats/.rem-cat`, `.mgr-group/.mgr-card/.mgr-t/.mt-num/.mgr-body/.mgr-guest/.mgr-why/.mgr-tags/.mgr-act/.mgr-empty/.me-ic`, `.live-tag/.ldot` | 795–1000, 916–952 |
| Seed data beyond the contract: guest **`g8`** (Kenji Watanabe), `prevIssue`, `tasteTags`, `seatingPref`, `t.remembered`, and **`t.sw` already seeded on tables 7, 11, 14, 15** | 2935–3053, 3054–3107 |
| `pol.courseJourney: p >= 3` (journey/pips are Phase-3 gated — they claim timing we cannot measure) | 3204 |

Everything below is written to **extend these exact functions**. Field names, class names and function names in this spec are the ones already in the file wherever one exists.

---

## 1. FINAL SIGNAL CATALOG

**106 signals, deduplicated from ~180 proposals across six catalogs.** 61 fire on today's data (`✔`), 45 are roadmap (`○`).

### 1.0 Trigger convention — the extended `signalContext`

Every `when` below is `c => …` against **one** context object. Extend `signalContext()` (3893) to return these; no trigger may read anything else. This is what keeps 106 triggers readable and keeps signals pure.

```js
function signalContext(t, pol) {
  const profiles = (t.party || []).map(g => guests[g]).filter(Boolean);
  const guest    = profiles[0] || null;
  const allergies = [...new Set([...(t.allergies||[]), ...profiles.flatMap(p=>p.allergies||[])])];
  const keys      = allergies.map(a => a.toLowerCase());
  const items = t.items || [], drinks = t.drinks || [], sw = t.sw || {};
  const stage = tableStage(t), pax = paxOf(t);
  const profOcc = profiles.find(p => p.occasion);
  const occasion = t.occasion || (profOcc && profOcc.occasion) || '';
  return {
    t, pol, guest, profiles, pax, stage, mins: t.seatedMin || 0, items, drinks, sw,
    now: nowMin,
    // dietary
    allergies, keys,
    severe: allergies.filter(a => profiles.some(p => (p.allergySeverity||{})[a] === 'anaphylaxis')),
    unsafe: items.filter(i => MENU_INDEX[i.name] && !isSafe(MENU_INDEX[i.name], keys)),
    unverifiable: allergies.filter(a => !Object.values(MENU).flat().some(m => (m.tags||[]).includes(a.toLowerCase()))),
    doorOnly: (t.allergies||[]).filter(a => !profiles.some(p => (p.allergies||[]).includes(a))),
    cartUnsafe: (sw.cartItems||[]).filter(n => MENU_INDEX[n] && !isSafe(MENU_INDEX[n], keys)),
    dislikes: [...new Set(profiles.flatMap(p => p.dislikes || []))],
    // occasion
    occasion, occType: occasionType(occasion),                      // 'Birthday'|'Anniversary'|'Engagement'|'Celebration'|''
    occasionName: (t.guestName || (profOcc&&profOcc.name) || (guest&&guest.name) || '').split(' ')[0],
    occasions: [...new Set([...(t.occasion?[t.occasion]:[]), ...profiles.map(p=>p.occasion).filter(Boolean)])],
    // relationship
    prevIssue: profiles.map(p => p.prevIssue).find(Boolean) || null,
    churn: pol.predictiveCues ? profiles.find(p => p.churnRisk) : null,
    gap: guest ? (guest.daysSinceLastVisit != null ? guest.daysSinceLastVisit : daysSince(guest.lastVisit)) : null,
    notes: profiles.map(p => p.notes).filter(Boolean),
    memory: profiles.flatMap(p => p.memory || []).concat(t.remembered || []),
    // order shape
    ap: courseQty(t,'apps'), mn: courseQty(t,'mains'), ds: courseQty(t,'dessert'),
    dr: drinkQty(t), bottle: hasBottle(t), ideal: idealCounts(pax),
    unserved: items.filter(i => !i.served),
    // service clocks (null when unmeasurable — a trigger MUST treat null as "do not fire")
    serveGap: t.lastServeMin ?? null, orderAge: t.lastOrderMin ?? null,
    touchGap: t.lastStaffTouchMin ?? null, attnAge: t.attentionMin ?? null,
    stageAge: t.stageEnteredMin ?? null, mgrGap: t.mgrVisitMin ?? null,
    bill: t.billTotal ?? null, paidAge: t.paidMin ?? null,
    reqAge: sw.requestPending ? (sw.requestMin || 0) : null,
    issue: t.issue || null, recovery: t.recovery || null,
    // floor context
    section: serverLoad(t.server), zone: zoneLoad(t.zone), staff: state.currentStaffName || null,
    actioned: id => isActioned(t.id, id),
  };
}
```

Column key: **tier**·**aud** (w=waiter, m=manager)·**sev** (R/A/N)·**pri** = catalog `score` (0–100, ranked *within* tier by §3). Copy cell is `title` · *why* · **action** → done.

---

### PILLAR 1 · SAFETY & DIETARY — red lives here and almost nowhere else

| id | tier | aud | sev | pri | trigger (`when: c =>`) | title · why · action → done | today |
|---|---|---|---|---|---|---|---|
| `allergy-brief` | attention | w | R | 100 | `c.allergies.length && c.items.length && !c.t.allergyBriefedAt && c.t.status!=='paid'` | **Kitchen not briefed on allergy** · *`${allergies.join(' & ')}` at this table and `${items.length}` plates are already in — the brief has not been logged.* · **Brief the kitchen** → Kitchen briefed | ✔ |
| `unsafe-plate` | attention | w | R | 99 | `c.unsafe.length > 0` | **Unsafe plate on this table** · *`${unsafe[0].name}` carries `${tag}` and `${allergies}` is flagged here.* · **Pull it and tell the kitchen** → Kitchen told | ○ needs MENU tag vocab + `items[].menuId` |
| `basket-allergy-conflict` | attention | w | R | 97 | `c.cartUnsafe.length > 0` | **Basket item clashes with their allergy** · *Their unsent basket has `${cartUnsafe[0]}` and `${allergies}` is flagged. The kitchen has not seen it yet.* · **Stop them before they send it** → Guest warned | ○ `sw.cartItems` |
| `allergy-unverifiable` | attention | w | A | 95 | `c.unverifiable.length > 0` | **This allergy can't be checked against the menu** · *`${unverifiable}` is on file but no dish is tagged for it — the safe list on this screen does not cover it.* · **Verify with the kitchen by hand** → Verified by hand | ✔ |
| `allergy-door-only` | attention | w | A | 90 | `c.doorOnly.length > 0` | **Allergy noted at seating, not on the profile** · *`${doorOnly}` was captured when the table opened but sits on no profile — suggestions are not filtering for it.* · **Save it to the guest profile** → Saved to profile | ✔ |
| `allergen-checking` | attention | w | A | 86 | `(c.sw.allergenViews||0)>=2 && c.allergies.length` | **Guest is checking allergen info** · *They opened allergen details `${sw.allergenViews}`× — last on `${sw.allergenItem}`. They want certainty before they order.* · **Confirm that dish with the kitchen** → Kitchen confirmed | ○ `sw.allergenViews` |
| `dietary-undeclared` | attention | w | A | 82 | `(c.sw.allergenViews||0)>=1 && !c.allergies.length` | **Dietary need not on file** · *They are reading allergen info and we hold nothing for this table.* · **Ask and add it to the table** → Added to table | ○ `sw.allergenViews` |
| `guest-dislikes` | context | w | N | 40 | `c.dislikes.length` | *Avoid `${dislikes}`* → renders as a `.ctx-chip`, never a card | ✔ |
| `dislike-on-ticket` | opportunity | w | A | 55 | `c.unserved.some(i => (MENU_INDEX[i.name]?.flavourTags||[]).some(tg => c.dislikes.some(d=>d.toLowerCase().includes(tg))))` | **Something on the ticket they don't like** · *`${dislikes[0]}` is on file and an unfired dish matches it.* · **Check with the table** → Checked | ○ `MENU.flavourTags` |

*Merged:* `allergy-at-table`+`allergy-brief-unconfirmed` → `allergy-brief`. `ordered-dish-clashes-allergy`+`unsafe-plate-on-table` → `unsafe-plate`. `allergen-info-checked`→`allergen-checking`; `undisclosed-dietary-need`→`dietary-undeclared`.

---

### PILLAR 2 · LIVE SERVICE FAILURE — measurable, never inferred

| id | tier | aud | sev | pri | trigger | copy | today |
|---|---|---|---|---|---|---|---|
| `guest-request` | attention | w | R | 100 | `c.sw.requestPending && (c.sw.requestType\|\|'assistance')!=='check'` | **Guest needs assistance** · *They asked from the table `${sw.requestMin} min ago`: "`${sw.requestText}`".* · **Respond to the table** → Responded | ✔ *(already live, table 15)* |
| `check-waiting` | attention | w | R | 90 | `(c.sw.requestPending && c.sw.requestType==='check') \|\| (c.sw.checkRequestedMin\|\|0)>=3 && !c.t.progress[3]` | **They asked for the bill** · *Requested `${n} min ago` and the check hasn't gone down — the last minute is the one they remember.* · **Drop the check** → Check dropped | ○ `sw.requestType` / `sw.checkRequestedMin` |
| `order-submit-failed` | attention | w | R | 94 | `(c.sw.submitFails\|\|0)>=1 && (c.sw.cartCount\|\|0)>=1` | **Their order failed to send** · *It didn't go through `${sw.submitFails}`× — `${sw.cartCount}` items are on their phone and the kitchen has nothing.* · **Take the order manually now** → Order entered | ○ |
| `payment-failed` | attention | w | R | 92 | `c.sw.payFailed && c.t.status!=='paid'` | **Their payment did not go through** · *The last attempt failed and the bill is still open — they're sitting with a dead screen.* · **Take payment at the table** → Paid | ○ |
| `payment-stalled` | attention | w | A | 76 | `(c.sw.payStartedMin\|\|0)>=6 && c.t.status!=='paid' && !c.sw.payFailed` | **Stuck at payment** · *They opened payment `${n} min ago` and haven't finished — usually a card or a question.* · **Offer to take payment** → Paid | ○ |
| `check-unsettled` | attention | w | A | 62 | `c.t.progress[3] && (c.t.checkDroppedMin\|\|0)>=10 && c.t.status!=='paid' && !c.sw.payStartedMin` | **Check out `${n}`m, not settled** · *No payment started — they may be waiting for you to come back.* · **Return to close the bill** → Settled | ○ |
| `open-issue-tonight` | attention | w | R | 91 | `c.issue && !c.issue.resolved` | **Open issue on this table** · *`${issue.note}` was logged `${issue.loggedMin} min ago` and hasn't been closed out.* · **Make it right now** → Resolved | ○ `t.issue` |
| `recovery-followthrough` | attention | w | A | 64 | `c.recovery?.acknowledged && !c.recovery.closed && (c.t.progress[2]\|\|c.stage==='check')` | **Close out the recovery before they leave** · *You flagged extra care for `${recovery.reason}` — log what you actually did.* · **Log what you did** → Logged | ○ `t.recovery` |
| `course-overdue` | attention | w | R | 92 | `c.t.status==='overdue'` | **Table has been waiting** · *Seated `${mins} min` and still on `${STAGE_WORD[stage]}`.* · **Check the pass and update them** → Table updated | ✔ *(live)* |
| `course-gap` | attention | w | A→R | 88 | `c.serveGap!==null && c.serveGap>=18 && c.unserved.length && c.stage!=='check'` — **red at `>=30`** | **Nothing served for `${serveGap}`m** · *`${unserved.length}` ordered plates are still out.* · **Check the pass, then update them** → Table updated | ○ `t.lastServeMin` |
| `seated-no-order` | attention | w | A→R | 86 | `c.stage==='seated' && !c.items.length && !c.dr && c.mins>=8 && !c.sw.requestPending && !(c.sw.cat&&c.sw.catMin>=3) && !(c.sw.menuOpenMin>=4&&!c.sw.itemsViewed)` — **red at `mins>=14`** | **No order yet** · *Seated `${mins} min` with nothing ordered — they're waiting on you, not the menu.* · **Take their order** → Order taken | ✔ *(live; add red step)* |
| `needs-attention` | attention | w | A | 74 | `c.t.status==='attention'` | **Table needs a check-in** · *Seated `${mins} min`, progressing slowly through `${STAGE_WORD[stage]}`.* · **Check in with the table** → Checked in | ✔ *(live)* |
| `stage-behind-benchmark` | attention | w | A | 72 | `c.stageAge!==null && c.stageAge >= (venue.stageBenchmarkMin[c.stage]\|\|999)*1.5 && c.unserved.length && c.stage!=='check'` | **Still on `${stage}` well past normal** · *`${stageAge}`m on `${stage}` — tables here move on by `${bench}`m.* · **Move the next course** → Course moved | ○ `venue.stageBenchmarkMin` |
| `no-staff-touch` | attention | w | A | 68 | `c.touchGap!==null && c.touchGap>=15 && !['paid','empty'].includes(c.t.status) && c.stage!=='check'` | **Nobody has touched this table in `${touchGap}`m** · *No serve, order or note logged while they're mid-meal.* · **Do a table check** → Checked | ○ `t.lastStaffTouchMin` |
| `drinks-not-poured` | attention | w | A | 72 | `c.dr>0 && !c.t.drinkProgress[0] && c.orderAge!==null && c.orderAge>=8 && c.stage!=='check'` | **Drinks ordered `${orderAge}`m ago, none poured** · *`${dr}` drinks came through the QR and nothing is on the table.* · **Pour the first round** → Drinks poured | ○ `t.lastOrderMin` |
| `drinks-low` | attention | w | A | 66 | `c.drinks.length && (c.t.drinkAgeMin\|\|0)>=20 && !c.bottle && ['starters','mains'].includes(c.stage)` | **Glasses running low** · *The round was poured `${drinkAgeMin} min ago`.* · **Offer another round** → Round offered | ✔ *(live)* |
| `bottle-not-poured` | attention | w | A | 50 | `c.bottle && (c.t.drinkAgeMin\|\|0)>=15 && c.stage!=='check'` | **Bottle open, glasses not topped** · *Last pour `${drinkAgeMin}`m ago — pouring is ours to do, not theirs.* · **Top up the glasses** → Topped up | ✔ |
| `party-served-unevenly` | attention | w | A | 66 | `c.pax>=6 && c.serveGap>=6 && sameCourse(c).some(i=>i.served) && sameCourse(c).some(i=>!i.served)` | **Some plates down, some still waiting** · *Half the table is eating alone `${serveGap}`m in.* · **Get the rest of the course out together** → Course evened up | ○ `t.lastServeMin` |
| `no-drinks-ordered` | opportunity | w | A | 70 | `c.dr===0 && ['starters','mains'].includes(c.stage)` | **Nothing to drink on this table** · *Food is moving and not one drink is ordered across `${pax}` covers.* · **Offer drinks** → Offered | ✔ |
| `items-in-kitchen` | context | w | N | 45 | `c.unserved.length` | *`${unserved.length}` of `${items.length}` dishes still coming — `${names}`* → context bullet | ✔ |
| `ready-to-turn` | attention | w | N | 58 | `c.t.status==='paid'` | **Bill paid** · *The party has settled and the table is free to reset.* · **Clear and turn the table** → Table cleared | ✔ *(live)* |
| `table-state-baseline` | context | w | N | 30 | `!['empty','paid'].includes(c.t.status)` | *Party of `${pax}` · `${stage}` · seated `${mins}`m · `${t.server}`'s table* → the always-last context bullet | ✔ |
| `covering-another-server` | context | w | N | 28 | `c.staff && c.t.server && c.t.server!==c.staff` | *`${t.server}`'s table — you're covering, read the history before you speak* | ○ `state.currentStaffName` |

*Merged:* `guest-raised-request`+`guest-called-for-help`+`guest-request-open` → `guest-request`. `check-requested-digitally`+`check-requested-waiting` → `check-waiting`. `course-gap-long`+`course-gap-critical` → `course-gap` (one signal, severity steps). `seated-no-order`+`seated-no-order-yet`+`seated-no-order-critical` → `seated-no-order`. `table-flagged-overdue`→`course-overdue`; `table-flagged-attention`→`needs-attention`. `drinks-going-flat`+`drinks-round-aging` → `drinks-low`.

---

### PILLAR 3 · LIVE GUEST BEHAVIOUR (SmartWeb) — what the floor cannot see

| id | tier | aud | sev | pri | trigger | copy | today |
|---|---|---|---|---|---|---|---|
| `browsing-category` | opportunity | w | N | 88 | `c.sw.cat && c.sw.catMin>=3 && (c.sw.catAdds\|\|0)===0 && (c.sw.lastActivityMin\|\|0)<=2 && !(/wine/i.test(c.sw.cat)&&c.bottle)` | **Guest may need assistance** · *`${sw.catMin} min` on the `${cat}` list without ordering.* · **Offer help choosing `${a wine \| from ${cat}}`** → Assistance offered · *wine variant action:* **Send the sommelier over** | ✔ *(live, table 11)* |
| `repeated-item` | opportunity | w | N | 76 | `c.sw.repeatItem && c.sw.repeatViews>=3 && !c.sw.repeatItemAdded && !c.items.some(i=>i.name===c.sw.repeatItem)` | **Undecided on a dish** · *They've opened `${repeatItem}` `${repeatViews}`× without adding it.* · **Talk them through the `${repeatItem}`** → Dish explained · *when `sw.repeatItemPremium`:* **Recommend it in person** ("the nudge usually lands here") | ✔ *(live, table 14; premium split ○)* |
| `ordering-stalled` | attention | w | A | 80 | `c.sw.menuOpenMin>=4 && (c.sw.itemsViewed\|\|0)===0 && !c.items.length` | **Ordering hasn't started** · *The menu has been open `${menuOpenMin} min` with nothing opened yet.* · **Offer to walk them through the menu** → Walked through | ✔ *(live, table 7)* |
| `menu-gone-cold` | attention | w | A | 79 | `(c.sw.itemsViewed\|\|0)>=1 && !(c.sw.cartCount\|\|0) && (c.sw.lastActivityMin\|\|0)>=5 && !c.items.length && c.mins>=8` | **Table went quiet on the menu** · *They looked at `${itemsViewed}` dishes, stopped `${lastActivityMin} min ago`, ordered nothing.* · **Take the order at the table** → Order taken | ✔ |
| `cart-stalled` | attention | w | A | 80 | `(c.sw.cartCount\|\|0)>=1 && (c.sw.cartMin\|\|0)>=3 && !c.sw.requestPending` | **Order sitting unsent in their basket** · *`${cartCount}` items untouched for `${cartMin} min``${cartValue? ' ($'+cartValue+')':''}`.* · **Check in and take it verbally** → Order taken | ○ `sw.cart*` |
| `ordering-hesitation` | attention | w | A | 76 | `!(c.sw.cartCount\|\|0) && (c.sw.itemsViewed\|\|0)>=3 && c.sw.menuOpenMin >= swBench.medianDecideMin*2 && !c.items.length && (c.sw.lastActivityMin\|\|0)<=2` | **Taking much longer than usual to order** · *`${menuOpenMin} min` with nothing chosen — tables here decide in `${swBench.medianDecideMin} min`.* · **Step in and guide the order** → Order guided | ○ `swBench` |
| `no-menu-scanned` | attention | w | A | 70 | `!c.t.sw && c.t.status==='seated' && c.mins>=5 && !c.items.length && !c.drinks.length` | **No one has opened the menu** · *Seated `${mins} min` and no phone here has scanned the QR.* · **Bring a menu and show the QR** → Menu given | ✔ |
| `dessert-menu-browsing` | opportunity | w | N | 66 | `c.sw.cat==='Desserts' && (c.sw.catMin\|\|0)>=1 && c.ds===0 && c.t.progress[1] && !c.t.progress[3]` | **Table is reading the dessert menu** · *Mains are done and nothing sweet is ordered — they're open to being closed.* · **Take dessert and coffee now** → Dessert taken | ✔ |
| `single-device-large-party` | opportunity | w | N | 68 | `c.pax>=5 && (c.sw.orderingDevices\|\|0)===1 && (c.sw.devices\|\|1)<=2 && ((c.sw.cartCount\|\|0)>=1 \|\| c.items.length)` | **One phone ordering for the whole table** · *Party of `${pax}` and one device — the rest of the table is being missed.* · **Share the QR around the table** → QR shared | ○ |
| `abandoned-intent` | opportunity | w | N | 64 | `c.sw.abandonedItem && (c.sw.abandonedMin\|\|99)<=4 && !c.items.some(i=>i.name===c.sw.abandonedItem)` | **Dropped something they wanted** · *`${abandonedItem}` was added then removed `${abandonedMin} min ago` — usually price or portion doubt.* · **Offer it a different way** → Offered | ○ |
| `menu-reopened` | opportunity | w | N | 58 | `c.sw.reopenedMin!=null && c.sw.reopenedMin<=3 && !(c.sw.cartCount\|\|0) && c.items.length && !c.t.progress[3]` | **Table is back on the menu** · *Reopened `${reopenedMin} min` after ordering — a second round is on their mind.* · **Offer another round** → Round offered | ○ |
| `menu-search-no-match` | opportunity | w | N | 56 | `c.sw.searchTerm && c.sw.searchNoResult` | **Searched for something we don't list** · *They searched "`${searchTerm}`" and got nothing — the kitchen can often work around it.* · **Offer the closest thing we do** → Alternative offered | ○ |
| `browsing-now` | context | w | N | 32 | `c.t.status!=='paid' && (c.sw.lastActivityMin\|\|9)<=2 && (c.sw.itemsViewed\|\|0)>=1` | *On the menu now — `${cat}`, `${itemsViewed}` dishes viewed, last tap `${lastActivityMin}`m ago* | ✔ |

*Merged:* `browsing-one-category`+`browsing-category`+`wine-list-dwell`+`extended-category-browse` → **one** `browsing-category` with a wine copy branch. `repeat-viewing-premium-item`+`repeated-item`+`repeat-item-hesitation`+`premium-item-revisited` → **one** `repeated-item` with a premium branch. `menu-open-nothing-viewed`+`menu-open-no-activity` → `ordering-stalled`. `order-stalled-in-basket`+`order-stalled-in-cart` → `cart-stalled`.

---

### PILLAR 4 · RELATIONSHIP, RECOGNITION & RECOVERY

| id | tier | aud | sev | pri | trigger | copy | today |
|---|---|---|---|---|---|---|---|
| `prev-service-issue` | attention | w | R | 96 | `!!c.prevIssue` | **Previous service issue** · *`${prevIssue.when}`: `${prevIssue.note}` · rated `${prevIssue.rating}★`. Get the first two minutes right and it's forgotten.* · **Prioritise this table** → Table prioritised | ✔ *(live, g7)* |
| `ratings-slipping` | attention | w | A | 70 | `c.churn && !c.prevIssue` | **Ratings have been slipping** · *`${first}`'s last three visits averaged `${avg}★`, and it's been `${gap}` days.* · **Check in early and often** → Checking in early | ✔ *(live)* |
| `low-last-rating` | attention | w | A | 60 | `c.guest && lastRating(c.guest)<=3 && !c.churn && !c.prevIssue` | **Last visit rated `${r}★`** · *They came back anyway — earn it tonight.* · **Check in after the first course** → Checked in | ✔ |
| `churn-risk-return` | attention | w | A | 78 | `c.churn && c.mins<=25` | **We nearly lost this guest** · *`${first}` is at risk after `${lastVisit}` away with ratings sliding — tonight decides whether they come back.* · **Give them your best service tonight** → Looked after | ✔ |
| `pacing-note` | attention | w | A | 68 | `c.notes.some(n=>/pacing\|gaps between courses/i.test(n)) \|\| (c.guest?.lastFeedback?.rating<=3 && c.guest.lastFeedback.tags?.length)` | **`${tag}` was the complaint last time** · *Fix that one thing and the visit reads as fixed.* · **Keep courses moving, no dead time** → Pace held | ✔ (`notes`) / ○ (`lastFeedback.tags`) |
| `staff-note` | context | w | N | 44 | `c.notes.length` | *`${notes.join(' · ').replace('⚠ ','')}* → context bullet, read before you approach | ✔ |
| `returning-guest` | opportunity | w | N | 72 | `c.guest && c.guest.visits>=1 && c.mins<=12` | **Returning guest** · *`${first}` has dined here `${visits}` times, last `${lastVisit}`, `${avgRating}★` average. Using the name in the first 60 seconds is the whole play.* · **Welcome them back by name** → Welcomed back | ✔ *(live)* |
| `loyal-regular` | context | w | N | 50 | `c.guest && c.guest.visits>=5` | *A regular — `${visits}` visits`${gap<=7?', in only '+gap+' days ago — offer something other than last time':''}`. Recognition beats explanation.* | ✔ |
| `first-visit` | opportunity | w | N | 54 | `c.guest && !c.guest.visits && !(c.guest.networkVisits\|\|0)` | **First time with us** · *No history anywhere — everything tonight sets the baseline.* · **Walk them through the menu** → Walked through | ✔ |
| `habit-forming` | opportunity | w | N | 48 | `c.guest && c.guest.visits>=1 && c.guest.visits<=2` | **Still deciding about us** · *Visit `${visits}` — the habit either forms now or it doesn't.* · **Make one thing memorable** → Done | ✔ |
| `network-first-visit` | opportunity | w | N | 68 | `c.pol.showNetworkBadge && c.guest && !c.guest.visits && (c.guest.networkVisits\|\|0)>=5` | **Known guest, first time here** · *`${first}` has dined `${networkVisits}` times across the group`${networkTier?', '+networkTier+' tier':''}` but never here. Treat them as a regular, not a stranger — and don't claim history in this room.* · **Introduce the room properly** → Room introduced | ✔ *(live)* |
| `long-absence` | opportunity | w | N | 64 | `c.guest && c.guest.visits>0 && c.gap>=60 && !c.churn && c.mins<=15` | **First visit in a while** · *`${first}` last dined here `${gap} days ago` after `${visits}` visits. Naming the gap reads as memory; ignoring it reads as forgetting.* · **Make them feel remembered** → Guest welcomed back | ✔ |
| `high-rated-relationship` | context | w | N | 38 | `c.guest && avgRating(c.guest)>=4.7` | *`${avgRating}★` over `${n}` visits — hold the standard, don't reinvent it* | ✔ |
| `review-ask` | opportunity | w | N | 32 | `c.t.status==='paid' && c.guest && avgRating(c.guest)>=4.5` | **Good night here — ask for the review** · *Asked in person it converts; the QR receipt alone mostly doesn't.* · **Ask on the way out** → Asked | ✔ |
| `taste-profile` | context | w | N | 36 | `(c.guest?.tasteTags\|\|[]).length` | *Leans `${tasteTags.join(' · ')}` — use it for the off-menu and the wine talk* | ✔ |
| `seating-pref-known` | context | w | N | 34 | `!!c.guest?.seatingPref` | *Prefers `${seatingPref}` — note it for the next booking* | ✔ |
| `multiple-known-guests` | context | w | N | 42 | `c.profiles.length>=2` | *`${names}` are all on file and their flags differ — recommend per seat, not per table* | ✔ |
| `partially-identified` | context | w | N | 40 | `c.t.party.length>0 && c.pax>c.t.party.length` | *`${party.length}` of `${pax}` guests on file — what you see covers `${party.length}`, not the table* | ✔ |
| `large-party-rhythm` | context | w | N | 38 | `c.pax>=6` | *`${pax}` covers: courses land together or not at all, and the check usually splits* | ✔ |
| `door-note` | context | w | N | 46 | `!!c.t.note` | *Host wrote: `${t.note}`* | ✔ |
| `remembered-note` | context | w | N | 48 | `c.memory.some(m=>m.scope!=='visit' && ['preference','conversation','seating','other'].includes(lc(m.cat)) && m.lastSurfacedVisit!==c.t.seatCycle)` | **Remembered from last visit** · *"`${m.value}`" — remembered by `${m.byServer}`, `${m.ts}`.* · **Got it** → Noted | ✔ (`t.remembered`) / ○ (attribution fields) |
| `memory-stale-confirm` | context | w | N | 22 | `c.memory.some(m=>m.scope==='guest' && m.surfacedCount>=3 && !m.confirmedCount)` | **Still true?** · *"`${m.value}`" has come up `${n}`× without ever being confirmed.* · **Confirm or remove** → Updated · *briefing only, never the floor* | ○ |
| `memory-conflict` | context | w | N | 20 | `conflictingMemory(c)` | **This changed since last time** · *"`${old.value}`" was saved as `${old.polarity}` by `${old.byServer}`; the newest note says `${next.polarity}`.* · **Pick the right one** → Resolved | ○ |
| `capture-guest` | opportunity | w | N | 34 | `!['empty','paid'].includes(c.t.status) && c.pax>c.t.party.length && c.mins>=20 && c.stage!=='check'` | **Most of this table is anonymous** · *`${pax}` covers, `${party.length}` recognised. One phone at the check makes their next visit personal.* · **Add guest info** → Info added | ✔ |
| `remember-first-timer` | opportunity | w | N | 58 | `(!c.guest \|\| !c.guest.visits) && ['dessert','check'].includes(c.stage) && !(c.t.memoryAddedCount\|\|0)` | **Nothing on file for this table yet** · *First visit for `${t.guestName\|\|'this party'}` — one detail now makes their next visit feel known.* · **+ Remember something** → Saved to profile | ✔ |
| `remember-observed-usual` | opportunity | w | N | 52 | `c.guest && c.items.some(i=>(c.guest.orderHistoryItems\|\|[]).some(h=>h.name===i.name&&h.visits>=2) && !c.guest.favFood.includes(i.name))` | **`${item.name}` every visit — not saved yet** · *Ordered on `${h.visits}` visits but not in their usuals.* · **Save as their usual** → Added to usuals | ○ `orderHistoryItems` |

*Merged:* `named-guest-recognition`+`returning-guest`(×2) → `returning-guest`. `loyal-regular`+`frequent-recent-regular` → `loyal-regular` (copy branch on `gap<=7`). `lapsed-guest-returning`+`long-absence`(×2)+`lapsed-return-welcome` → `long-absence`. `rating-declining`+`declining-ratings`+`ratings-slipping` → `ratings-slipping`; `long-absence-declining` folded into `churn-risk-return`. `feedback-theme-care` → `pacing-note`. `anonymous-table-no-profile`+`capture-covers` → `capture-guest`. `memory-context-surface`+`remembered-note-honour` → `remembered-note`.

---

### PILLAR 5 · OCCASION

| id | tier | aud | sev | pri | trigger | copy | today |
|---|---|---|---|---|---|---|---|
| `occasion-setup` | opportunity | w | N | 84 | `c.pol.occasionInBanner && c.occasion && !c.t.occasionMarkedAt && c.mins<=30 && c.stage!=='check'` | **`${occType}` tonight — set the moment up** · *On file for `${occasionName}` · party of `${pax}`. The kitchen needs the candle/plating note before dessert is fired, not when it's ordered.* · **Brief the kitchen now** → Kitchen briefed | ✔ *(live as `occasion`)* |
| `occasion-toast` | opportunity | w | N | 69 | `c.pol.occasionInBanner && /engagement\|celebration/i.test(c.occasion) && ['seated','starters'].includes(c.stage) && c.dr===0` | **Celebration at this table — open with a toast** · *`${occasion}` for `${occasionName}`, nothing poured yet`${suggestedDrink?' · they drink '+name:''}`.* · **Offer a toast pour** → Toast offered | ✔ |
| `occasion-last-chance` | attention | w | A | 88 | `c.pol.occasionInBanner && c.occasion && !c.t.occasionMarkedAt && (c.stage==='dessert' \|\| c.stage==='check' \|\| (c.t.progress[1] && !c.t.progress[2] && c.ds===0))` | **`${occType}` still unmarked** · *They're at `${stage}` with no gesture logged — dessert is the last moment we get.* · **Send the gesture now** → Gesture sent | ✔ |
| `two-occasions` | attention | w | A | 56 | `c.pol.occasionInBanner && c.occasions.length>=2` | **Two occasions at one table** · *`${occasions.join(' and ')}` are both here — confirm which they want marked.* · **Confirm which one to mark** → Confirmed | ✔ |
| `mgr-occasion` | opportunity | m | N | 74 | `c.pol.predictiveCues && c.occasion && c.profiles.some(g=>(g.spend\|\|0)>=3\|\|g.visits>=5) && c.stage!=='check' && !c.t.occasionMarkedAt` | **Occasion worth marking** · *`${occasionName}` is celebrating — `${visits}` visits, avg $`${avgPerCover}`. A comped `${suggestedDessert\|\|'dessert'}` is the cheapest loyalty we buy all night.* · **Approve it on the house** → Sent to the table | ✔ *(live)* |

*Merged:* `occasion-birthday`+`occasion-anniversary`+`occasion-at-table`+`occasion` → **one** `occasion-setup`; type only changes copy and icon (`ICONS`/`OCCASION_GLYPH`). `occasion-window-closing`+`occasion-plating-cue`+`occasion-last-chance` → **one** `occasion-last-chance`. `occasion-vip-unmarked`+`mgr-occasion` → `mgr-occasion`.

---

### PILLAR 6 · ORDER-GROUNDED OPPORTUNITY (the check, honestly earned)

| id | tier | aud | sev | pri | trigger | copy | today |
|---|---|---|---|---|---|---|---|
| `usual-drink-open` | opportunity | w | N | 66 | `c.stage==='seated' && c.dr===0 && c.mins<=20 && c.profiles.some(g=>g.suggestedDrink)` | **Open with the drink they always order** · *`${suggestedDrink.name}` — `${reason}`. Nothing poured yet, `${mins}`m in.* · **Offer the `${name}`** → Offered | ✔ |
| `usual-order` | opportunity | w | N | 60 | `c.guest && (c.guest.favFood\|\|[]).length && ['seated','starters'].includes(c.stage) && !c.items.some(i=>c.guest.favFood.includes(i.name))` | **They have a usual** · *`${first}` usually orders `${favFood.slice(0,2).join(' and ')}` and none of it is on tonight's ticket.* · **Lead with their usual** → Usual suggested | ✔ *(live)* |
| `usual-starter-rec` | opportunity | w | N | 48 | `['seated','starters'].includes(c.stage) && c.ap<c.ideal.starters && safeSuggestion(c,'suggestedStarter','starters')` | **Starter they'll say yes to** · *`${name}` — `${reason}`. `${ap}` of `${ideal.starters}` starters in`${allergyLabel}`.* · **Suggest the `${name}`** → Suggested | ✔ |
| `usual-main-rec` | opportunity | w | N | 54 | `!['seated','check'].includes(c.stage) && c.mn<c.ideal.mains && safeSuggestion(c,'suggestedMain','mains')` | **Main they order every time** · *`${name}` — `${reason}`. `${mn}` of `${ideal.mains}` mains in.* · **Suggest the `${name}`** → Suggested | ✔ |
| `usual-dessert-window` | opportunity | w | N | 60 | `c.t.progress[1] && !c.t.progress[3] && c.ds===0 && safeSuggestion(c,'suggestedDessert','desserts')` | **Dessert window is open** · *`${first}` finishes with `${name}` — `${reason}`. Mains cleared, nothing sweet ordered.* · **Offer the `${name}`** → Dessert offered | ✔ |
| `wine-pairing-mains` | opportunity | w | N | 61 | `c.mn>0 && !c.bottle && ['starters','mains'].includes(c.stage) && c.profiles.some(g=>g.suggestedDrink?.type==='wine')` | **Mains ordered, no wine on the table** · *`${mainName}` is in and no bottle open. `${first}` drinks `${wine}` — `${reason}`.* · **Offer the `${wine}`** → Wine offered | ✔ |
| `bottle-over-glasses` | opportunity | w | N | 53 | `!c.bottle && c.dr>=c.pax && c.pax>=3 && ['starters','mains'].includes(c.stage)` | **By the glass is adding up** · *`${dr}` glasses for `${pax}` covers, no bottle open. A bottle is better value and better service.* · **Offer the bottle** → Bottle offered | ✔ |
| `reserve-list-offer` | opportunity | w | N | 46 | `!c.bottle && ['starters','mains'].includes(c.stage) && c.profiles.some(g=>(g.avgPerCover\|\|0)>=2500 && (g.spend\|\|0)>=4)` | **This table buys at the top of the list** · *`${first}` averages $`${avgPerCover}` a cover across `${visits}` visits — the house pour isn't their ceiling.* · **Bring the reserve list** → Reserve list brought | ✔ |
| `menu-pairing-walkin` | opportunity | w | N | 44 | `c.mn>0 && !c.bottle && !c.profiles.length && c.items.some(i=>(MENU_INDEX[i.name]?.pairsWith\|\|[]).length)` | **Pair what they actually ordered** · *`${mainName}` is on the table, no wine open, no profile here — the order is the only signal we have and it's enough.* · **Offer the house pairing** → Pairing offered | ○ `MENU.pairsWith` |
| `digestif-close` | opportunity | w | N | 49 | `c.stage==='dessert' && !c.t.drinkProgress[2] && c.profiles.some(g=>g.suggestedDrink?.type==='spirit')` | **They always finish with a digestif** · *`${first}` orders `${name}` after the meal — `${reason}`.* · **Offer the `${name}`** → Offered | ✔ |
| `upsell` | opportunity | w | N | 52 | `c.guest?.upsell && c.items.length && c.stage!=='check' && (c.guest.upsellAffinity==null\|\|c.guest.upsellAffinity>=0.5)` | **Something they usually add** · *`${upsell.reason}``${affinity?' · accepts '+pct+'% of suggestions':''}`.* · **Mention the `${upsell.name}`** → Mentioned | ✔ *(live)* |
| `usual-ordered-say-so` | opportunity | w | N | 36 | `c.mins<=35 && c.guest && c.items.some(i=>(c.guest.favFood\|\|[]).includes(i.name))` | **They ordered their usual** · *`${item}` is on `${first}`'s ticket again. One line turns a repeat order into being remembered.* · **Say we remembered** → Said at the table | ✔ |
| `taste-discovery-rec` | opportunity | w | N | 38 | `['seated','starters'].includes(c.stage) && discoveryMatch(c)` | **Something new they're likely to like** · *`${first}` orders `${tasteTags.join(' + ')}` and has never had `${dish}` — a match on their own pattern, not a guess.* · **Suggest it as something new** → Suggested | ○ `MENU.flavourTags` |

*Merged:* `known-favourites`+`usual-order` → `usual-order`. `usual-drink-not-ordered`+`usual-drink-open` → `usual-drink-open`. `high-value-cover`+`reserve-list-offer` → `reserve-list-offer`.

---

### PILLAR 7 · MANAGER — "where do my next 10 minutes go"

| id | tier | aud | sev | pri | trigger | copy | today |
|---|---|---|---|---|---|---|---|
| `mgr-recovery` | attention | m | R | 98 | `c.prevIssue && c.guest && (c.guest.vipTier==='VIP'\|\|(c.guest.spend\|\|0)>=4\|\|(c.guest.visits\|\|0)>=5)` | **Recovery needs you** · *High-value guest whose last visit went wrong: `${prevIssue.note}`.* · **Visit the table personally** → Manager visited | ✔ *(live)* |
| `mgr-recovery-repeat` | attention | m | R | 99 | `c.prevIssue && (['attention','overdue'].includes(c.t.status) \|\| (c.stage==='mains'&&!c.t.progress[1]&&c.mins>=25))` | **Last visit's problem is repeating** · *`${name}` rated last visit `${rating}★` — `${note}` — and mains still aren't out `${mins}`m in.* · **Go to the table now** → Visited | ✔ |
| `mgr-multiple` | attention | m | R | 90 | `c._attnCount >= 2` | **Several signals at once** · *`${n}` things need attention on this table at the same time`${server?', and '+server+' is covering it alone':''}`.* · **Visit the table** → Manager visited | ✔ *(live)* |
| `mgr-overdue-vip` | attention | m | R | 96 | `c.t.status==='overdue' && c.profiles.some(g=>g.vipTier==='VIP'\|\|(g.spend\|\|0)>=4\|\|(g.avgPerCover\|\|0)>=2000)` | **Overdue table with a top guest on it** · *`${name}` — `${visits}` visits, avg $`${avgPerCover}` a head.* · **Own this table until it clears** → Cleared | ✔ |
| `mgr-request-unanswered` | attention | m | R | 95 | `c.reqAge!==null && c.reqAge>=5` | **Guest asked and is still waiting** · *"`${requestText}`" came from T`${id}` `${reqAge}`m ago and `${server}` hasn't cleared it.* · **Answer it at the table** → Answered | ✔ |
| `mgr-flag-unattended` | attention | m | R | 94 | `(c.attnAge>=8 && c.touchGap>=8) \|\| c.t.actions?.some(a=>a.severity==='red'&&a.status!=='done'&&(nowMin-a.detectedAtMin)>=12) \|\| ((nowMin-(c.t.lastViewedAtMin\|\|0))>=8 && c.t.actions?.some(a=>a.status==='detected'&&(nowMin-a.detectedAtMin)>=8))` | **Flagged table, nobody has been** · *T`${id}` has been flagged `${attnAge}`m and `${server}` hasn't touched it in `${touchGap}`m.* · **Cover it or reassign it** → Covered | ○ `attentionMin`, `lastStaffTouchMin`, `t.actions` |
| `mgr-approval-pending` | attention | m | A | 93 | `(c.t.openActions\|\|[]).some(a=>a.needsApproval && !a.done && (a.raisedMin\|\|0)>=2)` | **A gesture is waiting on your approval** · *`${server}` raised "`${label}`" for T`${id}` `${raisedMin}`m ago.* · **Approve or decline** → Decided | ○ `t.openActions` |
| `mgr-churn-guest` | attention | m | A | 92 | `c.churn && !c.prevIssue && c.mins<=45` | **At-risk regular is on the floor** · *`${name}` has `${visits}` visits but the last three ratings were `${r}★` — tonight decides whether they return.* · **Introduce yourself early** → Introduced | ✔ |
| `mgr-save-before-feedback` | attention | m | A | 91 | `['dessert','check'].includes(c.stage) && !c.t.progress[3] && (c.t.party.length\|\|c.t.guestRef) && (['attention','overdue'].includes(c.t.status) \|\| c.prevIssue \|\| c.churn \|\| (c.guest&&avgRating(c.guest)<4))` | **Last chance before the rating request** · *`${name}` is at `${stage}` after a rough service — the feedback request fires the moment they pay.* · **Visit before the check goes down** → Visited | ✔ |
| `mgr-no-server` | attention | m | A | 89 | `!['empty','paid'].includes(c.t.status) && !c.t.server && c.mins>=3` | **Seated table with no server on it** · *T`${id}` in `${zone}` has been seated `${mins}`m with nobody assigned.* · **Assign a server now** → Assigned | ✔ |
| `mgr-section-overload` | attention | m | A | 85 | `c.section.live>=5 && c.section.flagged>=2` | **`${server}`'s section is underwater** · *`${live}` live tables, `${flagged}` flagged, `${openActions}` open actions.* · **Take a table off this section** → Reassigned | ✔ |
| `mgr-zone-pressure` | opportunity | m | A | 80 | `c.zone.flagged>=2` | **One zone is where the trouble is** · *`${flagged}` of `${live}` live tables in `${zone}` are flagged.* · **Work this zone for ten minutes** → Worked it | ✔ |
| `mgr-vip-farewell` | attention | m | A | 88 | `c.t.status==='paid' && (c.paidAge==null\|\|c.paidAge<=6) && c.profiles.some(g=>g.vipTier==='VIP'\|\|(g.spend\|\|0)>=4\|\|g.visits>=5)` | **Top guest is about to walk out** · *T`${id}` has paid and `${name}` (`${visits}` visits, avg $`${avgPerCover}`) leaves in the next few minutes.* · **See them out yourself** → Said goodbye | ✔ |
| `mgr-vip` | opportunity | m | N | 78 | `c.guest && (c.guest.vipTier==='VIP'\|\|c.guest.spend===4) && c.guest.visits>0 && !c.prevIssue && c.mins<=25` | **Priority guest dining** · *`${first}` — `${visits}` visits, avg $`${avgPerCover}` a cover, rates us `${avgRating}★`. The highest-leverage 30 seconds on the floor tonight.* · **Personally welcome them** → Guest welcomed | ✔ *(live)* |
| `mgr-vip-first-visit` | opportunity | m | N | 93 | `c.profiles.some(g=>!g.visits && (g.vipTier==='VIP'\|\|(g.spend\|\|0)>=4\|\|g.networkTier==='platinum')) && c.mins<=30` | **First visit here for a top-tier guest** · *`${networkVisits}` visits across the group at avg $`${avgPerCover}` a head, none in this room.* · **Welcome them personally** → Welcomed | ✔ |
| `mgr-network-newcomer` | opportunity | m | N | 90 | `c.profiles.some(g=>!g.visits && (g.networkVisits\|\|0)>=5 && g.networkTier) && c.mins<=30` | **Group regular, first time in this room** · *`${networkTier}`-tier with `${networkVisits}` visits across the group and none here.* · **Introduce yourself and the room** → Introduced | ✔ |
| `mgr-regular-touch` | opportunity | m | N | 84 | `!['attention','overdue'].includes(c.t.status) && c.profiles.some(g=>g.visits>=5 && g.vipTier!=='VIP' && ((g.spend\|\|0)>=4\|\|(g.avgPerCover\|\|0)>=1800)) && ['starters','mains'].includes(c.stage)` | **Regular worth a minute of your time** · *Visit `${visits+1}` at avg $`${avgPerCover}` a head, settled into `${stage}`.* · **Stop by the table** → Stopped by | ✔ |
| `mgr-advocacy-chef` | opportunity | m | N | 82 | `c.profiles.some(g=>(g.advocacy\|\|0)>=70 \|\| g.networkTier==='platinum') && ['starters','mains'].includes(c.stage)` | **This guest brings you other guests** · *`${name}` keeps introducing new parties (`${networkVisits}` group visits, `${networkTier}`-tier) — worth more than tonight's bill.* · **Bring the chef out to them** → Chef went over | ✔ |
| `mgr-big-party` | opportunity | m | N | 83 | `c.pax>=6 && ['starters','mains'].includes(c.stage) && (c.mgrGap==null\|\|c.mgrGap>=30)` | **Big table, nobody senior has been** · *`${pax}` covers, `${mins}`m in at `${stage}` — big tables go wrong quietly.* · **Check in with the host of the table** → Checked in | ✔ (`mgrVisitMin` ○) |
| `mgr-high-cheque` | opportunity | m | N | 84 | `c.bill!==null && c.bill===maxOpenBill() && c.stage!=='check'` | **Biggest cheque on the floor** · *T`${id}` is at $`${bill}` across `${pax}` covers.* · **Check in before dessert** → Checked in | ○ `t.billTotal` |
| `mgr-seating-pref` | opportunity | m | N | 77 | `c.mins<=15 && c.profiles.some(g=>g.seatPref?.zone && g.seatPref.zone!==c.t.zone) && tables.some(x=>x.status==='empty'&&x.cap>=c.pax&&x.zone===prefZone(c))` | **Not the table they like** · *`${name}` usually sits `${seatPref.note}` and is in `${zone}` tonight — a free table matches.* · **Offer them the move** → Offered | ○ `guest.seatPref` |
| `mgr-untouched-tables` | context | m | N | 66 | `!['empty','paid'].includes(c.t.status) && c.mins>=40 && (c.mgrGap==null\|\|c.mgrGap>=40) && !['attention','overdue'].includes(c.t.status)` | **Tables nobody senior has been near** · *T`${id}` is `${mins}`m in with no manager visit all service.* · **Add one to your round** → Visited | ○ `t.mgrVisitMin` |
| `mgr-red-budget` | attention | m | A | 80 | `redShare() > 0.30` | **Too many tables flagged red** · *`${pct}`% of occupied tables are red — either the floor is short-handed or a flag is misfiring.* · **Review red flags** → Reviewed | ✔ |
| `mgr-shift-backlog` | attention | m | A | 74 | `openActionCount()>=8 \|\| shiftStats.medianActionLatencyMin>=8` | **Floor is behind on flagged tables** · *`${n}` actions open, median clear time `${m}`m.* · **Rebalance the floor** → Rebalanced | ○ `shiftStats` |
| `mgr-rating-exposure` | context | m | N | 62 | `flaggedIdentifiedCount()>=1` | **Ratings we'll be asked for tonight** · *`${identified}` of `${live}` live tables have an identified guest who gets a feedback request on payment — `${atRisk}` are flagged.* · **Open the flagged ones** → Reviewed | ✔ |
| `mgr-coverage-board` | context | m | N | 58 | `liveTableCount()>=6` | **Who is carrying what** · *`${perServerSummary}` — the fastest fix is usually moving one table.* · **Rebalance a section** → Rebalanced | ✔ |
| `mgr-occasion-load` | context | m | N | 56 | `occasionLoad()>=2` | **Occasions on tonight** · *`${n}` tables celebrating — confirm the gestures before the dessert wave.* · **Confirm the gestures with the pass** → Confirmed | ✔ |
| `mgr-first-timer-density` | context | m | N | 54 | `firstTimerTables()>=3` | **How much of the room is new** · *`${n}` of `${live}` live tables have never dined here.* · **Brief the floor before the next seating** → Briefed | ✔ |
| `mgr-table-finished-untouched` | context | m | N | 28 | `c.t.status==='paid' && attnRaised(c.t)>=2 && attnDone(c.t)===0` | **T`${id}` left with flags untouched** · *`${n}` attention items were raised (`${server}`) and none actioned before the check closed.* · **Review with server** → Reviewed | ✔ |
| `mgr-server-coaching-gap` | opportunity | m | N | 46 | `Object.entries(serverStats).some(([s,v])=>v.attentionSurfaced>=10 && v.doneRate<0.4 && shiftStats.floorDoneRate>0.7)` | **`${server}` is clearing `${x}` in 10 flagged actions** · *`${pct}`% vs a floor average of `${floorPct}`%.* · **Coach on the shift brief** → Coached | ○ `serverStats` |

**Door section (reservation-scoped — evaluated by `reservationSignals(r)`, not `tableSignals`):**

| id | tier | aud | sev | pri | trigger | copy | today |
|---|---|---|---|---|---|---|---|
| `door-recovery-arrival` | attention | m | A | 90 | `!r.seated && r.minutesUntil<=20 && r.guestId && (guests[r.guestId].prevIssue \|\| guests[r.guestId].churnRisk)` | **Recovery guest arriving shortly** · *`${r.name}` arrives in `${minutesUntil}`m — last visit ended at `${rating}★` (`${note}`).* · **Greet them at the door** → Greeted | ○ `r.minutesUntil` |
| `door-priority-arrival` | attention | m | A | 87 | `!r.seated && r.minutesUntil<=15 && r.guestId && (VIP\|\|spend>=4\|\|visits>=5\|\|r.occasion)` | **Priority arrival at the door** · *`${r.name}` in `${minutesUntil}`m for T`${r.table}` — `${visits}` visits`${occasion?', '+occasion:''}`.* · **Be at the door** → Greeted | ○ |
| `door-table-not-turning` | attention | m | A | 84 | `!r.seated && r.minutesUntil<=20 && assigned(r) && !['check','dessert'].includes(tableStage(assigned(r)))` | **Their table won't be free in time** · *`${r.name}` arrives in `${minutesUntil}`m for T`${r.table}`, still at `${stage}` after `${seatedMin}`m.* · **Re-seat them or hold the door** → Sorted | ○ |
| `res-occasion-prep` | opportunity | m | N | 52 | `pol.showReservations && r.occasion && !r.seated` | **Occasion booked tonight — prep before they sit** · *`${r.name}`, `${r.time}`, party of `${r.party}` · `${r.occasion}`. Prep now is invisible; prep at dessert is obvious.* · **Set the plating up** → Prepped | ✔ |
| `res-regular-brief` | opportunity | m | N | 51 | `pol.showReservations && r.guestId && guests[r.guestId].visits>=5 && !r.seated` | **Regular arriving — brief the section** · *`${r.name}` at `${r.time}` · `${visits}` visits, `${avgRating}★`, avg $`${avgPerCover}``${notes?' · '+notes:''}`.* · **Brief the section server** → Section briefed | ✔ |
| `res-commitment` | opportunity | m | N | 50 | `pol.showReservations && (r.commitments\|\|[]).length && !r.seated` | **We promised them something** · *Commitment for `${r.name}` (`${r.time}`): `${commitments.join(' · ')}`. It has to be ready before they sit and someone has to own it.* · **Assign it to someone** → Assigned | ○ `r.commitments` |
| `res-recurring-occasion` | opportunity | m | N | 62 | `r.guestId && (guests[r.guestId].occasionCalendar\|\|[]).some(o=>daysUntilCal(o)<=3)` | **`${o.label}` lands this week** · *`${r.name}` is booked `${r.time}` and their `${o.label}` is in `${daysUntil}` days — remembered by `${o.byServer}`.* · **Plan a gesture** → Gesture planned | ○ `occasionCalendar` |

---

### PILLAR 8 · THE ENGINE WATCHING ITSELF

| id | tier | aud | sev | pri | trigger | copy | today |
|---|---|---|---|---|---|---|---|
| `action-sla-breach` | attention | w | R | 96 | `c.t.actions?.some(a=>a.severity==='red'&&a.status==='seen'&&(nowMin-a.seenAtMin)>=5)` | **Urgent action still open** · *`${label}` has been open `${n}`m on this table.* · **Handle it now** → Handled | ○ `t.actions` |
| `action-pileup` | attention | w | A | 86 | `c.t.actions?.filter(a=>a.tier==='attention'&&['detected','seen'].includes(a.status)).length>=3` | **Three things stacked here** · *`${n}` items need attention — two are showing, the rest are behind +more.* · **Work the queue** → Queue cleared | ○ |
| `action-done-unverified` | attention | w | A | 72 | `c.t.actions?.some(a=>a.status==='done'&&a.verifiable&&(nowMin-a.doneAtMin)>=10&&a.triggerStillTrue)` | **Marked done, nothing changed** · *`${label}` was marked done `${n}`m ago but the table still looks the same.* · **Do it for real** → Sorted | ○ |
| `signal-noise-review` | context | m | N | 26 | `Object.entries(signalStats).some(([id,v])=>v.surfaced>=20 && v.dismissRate14d>0.6)` | **"`${title}`" is mostly being dismissed** · *Shown `${surfaced}`×, dismissed `${pct}`% over two weeks.* · **Turn it down** → Adjusted | ○ `signalStats` |
| `signal-high-yield` | context | m | N | 18 | `Object.entries(signalStats).some(([id,v])=>v.done>=15 && v.ratingLift>=0.3)` | **`${title}` is moving ratings** · *Tables where it was actioned rated `${lift}★` higher across `${done}` cases.* · **Share with the team** → Shared | ○ |

**Dropped as non-signals** (they are ranking *rules*, implemented in §3–§4, not catalog entries): P4 `experience-health`, P6 `red-budget-exceeded` duplicate (kept once as `mgr-red-budget`), P5 `server-coverage-board` duplicate of `mgr-coverage-board`, P6 `server-open-action-load` folded into `mgr-section-overload`.

---

## 2. NEW DATA FIELDS

Design rule applied: **one canonical home per fact.** Where two catalogs proposed the same fact under different names (`request` vs `sw.requestPending`; `lastVisitDays` vs `daysSinceLastVisit` vs `lastVisitAt`; `seatingPref` vs `seatingPrefZone` vs `seatPref`), exactly one survives — the one that matches names **already in the file**.

### 2.1 `table.sw` — the SmartWeb session (single namespace, absent when no phone has scanned)

Already partially seeded (tables 7, 11, 14, 15). Extend it; **keep the existing flat `requestPending / requestText / requestMin` names** — they are already consumed at 3928–3930, so no migration.

```js
// FULL shape — every field optional; absence is meaningful (no `sw` at all = nobody scanned)
sw: {
  // session
  menuOpenMin: 7,          // number, min since first scan at this table (max across devices)
  lastActivityMin: 0,      // number, min since any tap/scroll — gates "still deciding" vs "gone cold"
  itemsViewed: 6,          // number of distinct item detail views
  devices: 2,              // distinct open sessions
  orderingDevices: 1,      // sessions that added >=1 item
  // dwell
  cat: 'Wine',             // 'Wine'|'Starters'|'Mains'|'Desserts'|'Cocktails'|null — Title-Case, matches SmartWeb nav
  catMin: 4,               // continuous min in `cat`
  catAdds: 0,              // items added from `cat` this session
  // hesitation
  repeatItem: 'Wagyu Ribeye',
  repeatViews: 4,
  repeatItemAdded: false,
  repeatItemPremium: true, // top price quartile of its category → opportunity copy instead of attention
  abandonedItem: 'Osso Buco',
  abandonedMin: 2,
  searchTerm: 'gluten free pasta',
  searchNoResult: true,
  // basket
  cartCount: 3, cartMin: 6, cartValue: 184,
  cartItems: ['Truffle Pasta','Sea Bass','Tiramisu'],   // names must resolve through MENU_INDEX
  submitFails: 0,
  // dietary anxiety
  allergenViews: 2, allergenItem: 'Truffle Arancini',
  // explicit request  (canonical home — P4's `t.request` is NOT implemented)
  requestPending: true,
  requestType: 'wine-list', // 'assistance'|'wine-list'|'water'|'allergen'|'check'  (default 'assistance')
  requestText: 'Could we see the wine list?',
  requestMin: 2,
  // close
  checkRequestedMin: null,  // number|null — min since the guest asked for the bill
  payStartedMin: null,      // number|null — payment sheet opened, not completed
  payFailed: false,
  reopenedMin: null         // number|null — menu reopened after the last submitted order
}
```

**Concrete seed to add (drop-in, replaces table 14's `sw` and extends 11/15):**

```js
// table 11 — the flagship dwell case, now with adds/benchmark context
sw:{ cat:'Wine', catMin:4, catAdds:0, lastActivityMin:0, menuOpenMin:7, itemsViewed:5, devices:2, orderingDevices:0 },
// table 14 — premium hesitation + a stalled basket
sw:{ repeatItem:'Wagyu Ribeye', repeatViews:4, repeatItemAdded:false, repeatItemPremium:true,
     lastActivityMin:1, menuOpenMin:9, itemsViewed:7, cartCount:2, cartMin:6, cartValue:168,
     cartItems:['Wagyu Ribeye','Truffle Arancini'], allergenViews:0, devices:1, orderingDevices:1 },
// table 15 — explicit request, now typed
sw:{ requestPending:true, requestType:'wine-list', requestText:'Could we see the wine list?',
     requestMin:2, lastActivityMin:2, menuOpenMin:3, itemsViewed:2, devices:3, orderingDevices:1 },
// table 4 — new: dietary anxiety on a gluten table (drives allergen-checking)
sw:{ allergenViews:3, allergenItem:'GF Risotto', lastActivityMin:4, menuOpenMin:22, itemsViewed:11 },
```

`sw` **must be added to the `doMoveTable` transfer key list** (4697) — the session belongs to the party, not the room.

### 2.2 `table.*` — staff-side clocks and done-effects

| field | type | example | why |
|---|---|---|---|
| `seatCycle` | number | `3` | incremented by `confirmSeat` and `doMoveTable`; scopes action keys and memory writes to a **party**, not a table |
| `lastServeMin` | number | `22` | min since any `items[].served` / `drinkProgress` mark — from Pulse's own serve marks, not kitchen systems |
| `lastOrderMin` | number | `8` | min since the last QR order submission |
| `lastStaffTouchMin` | number | `16` | min since **any** staff action in Pulse on this table |
| `attentionMin` | number | `11` | min since the table entered `attention`/`overdue` — status carries no age today |
| `stageEnteredMin` | number | `26` | min in the current `tableStage(t)` |
| `mgrVisitMin` | number\|undefined | `undefined` | undefined = no manager visit this service |
| `billTotal` | number | `842` | running cheque |
| `paidMin` | number | `3` | min since payment cleared (farewell window ≈6 min) |
| `checkDroppedMin` | number\|null | `12` | min since `progress[3]` was marked |
| `lastViewedAtMin` | number | `41` | written by `openTableModal` — separates "ignored" from "never looked" |
| `allergyBriefedAt` | number\|null | `38` | **done-effect** of `allergy-brief` |
| `occasionMarkedAt` | number\|null | `null` | **done-effect** of `occasion-setup` / `mgr-occasion` |
| `memoryAddedCount` | number | `1` | Remember-This saves this `seatCycle` |
| `cardsSurfacedCount` | number | `4` | novelty cap counter (max 6 primaries per seating) |
| `actions` | ActionInstance[] | see §8 | the lifecycle store |
| `issue` | `{type,note,loggedMin,by,resolved}` | `{type:'pacing',note:'Starters landed cold',loggedMin:7,by:'Maria',resolved:false}` | a failure logged **tonight** |
| `recovery` | `{reason,owner,managerNotified,gesture,acknowledged,closed,closedBy}` | `{reason:'25-min course gap',owner:'James',managerNotified:true,gesture:'Comped dessert',acknowledged:true,closed:false}` | recovery workflow state |
| `openActions` | `[{id,label,audience,raisedMin,needsApproval,done}]` | `[{id:'mgr-occasion',label:'Comp the panna cotta',audience:'manager',raisedMin:4,needsApproval:true}]` | waiter→manager approval queue |

Add to `doMoveTable`'s key list: `'sw','seatCycle','actions','openActions','issue','recovery','allergyBriefedAt','occasionMarkedAt','lastServeMin','lastOrderMin','lastStaffTouchMin','attentionMin','stageEnteredMin','billTotal','memoryAddedCount','cardsSurfacedCount','remembered'`.

### 2.3 `guests[gid].*`

| field | type | example | why |
|---|---|---|---|
| `daysSinceLastVisit` | number | `78` | **the one canonical recency number** (kills `lastVisitDays`, `lastVisitAt`, `daysSince()` regex fragility). `lastVisit` string stays for display only |
| `visitDates` | ISO date[] | `['2026-06-14','2026-07-02']` | cadence copy ("always Fridays"), trend detection |
| `prevIssue` | `{when,rating,note}` | *already on g7* | the grounded recovery fact |
| `serviceIssues` | `[{date,type,summary,resolved,gesture}]` | `[{date:'2026-06-14',type:'pacing',summary:'25 min between courses',resolved:true,gesture:'Comped dessert'}]` | the historical half of recovery |
| `lastFeedback` | `{rating,tags[],comment,date}` | `{rating:3,tags:['pacing'],comment:'Slow',date:'2026-06-14'}` | turns "unhappy" into "fix this one thing" |
| `allergySeverity` | `Record<label,'preference'\|'intolerance'\|'anaphylaxis'>` | `{Nuts:'anaphylaxis'}` | `['Nuts','Vegetarian']` render identically today; severity decides red vs amber |
| `seatPref` | `{zone,tables[],note}` | `{zone:'patio',tables:[5,12],note:'Patio, away from the bar'}` | the prose `seatingPref` stays as the human note; `seatPref.zone` is what a trigger can compare |
| `memory` | MemoryEntry[] | see §8.4 | canonical staff-taught store (supersedes the flat `guest.remembered`) |
| `occasionCalendar` | `[{month,day,label,byServer}]` | `[{month:3,day:12,label:'Anniversary',byServer:'Maria'}]` | "every year" occasions |
| `occasionType` / `occasionDate` | enum / ISO | `'Anniversary'` / `'2026-09-01'` | `occasion:'Anniversary tonight'` mixes type and timing; separating stops "next month" firing a plating cue |
| `orderHistoryItems` | `[{name,visits,lastVisit}]` | `[{name:'Wagyu Ribeye',visits:4,lastVisit:'2026-08-13'}]` | evidence of repetition — powers `remember-observed-usual` and real reason strings |
| `upsellAffinity` | number 0–1 **on every guest** | `0.78` | present on g1 only today; without it the upsell card fires equally at guests who always decline |
| `advocacy` / `partiesBrought` | number / number | `87` / `6` | countable advocacy ranks advocates against each other |
| `lastServer` | string | `'James'` | "James looked after you last time" is the cheapest recognition lever we have |
| `language` | BCP-47 | `'ja-JP'` | already known from the SmartWeb session; changes who takes the table |
| `favFoodFreq` | `Record<name,{orders,visits}>` | `{'Lamb Shank':{orders:3,visits:4}}` | lets every rec state its own evidence instead of a hand-written reason |

**Deferred (documented, not built in v2):** `tables[].partySeats`, `tables[].hostGuestId`, `guests[].suggested*.confidence`. All three are correct but none unblocks a v2 signal; per-seat allergy routing needs a kitchen integration to be worth the model change.

### 2.4 Globals

```js
let nowMin = 0;                        // service clock, ticks with updateClock() (5452). ALL ages measured against it.
const MENU_INDEX = {};                 // built once: Object.values(MENU).flat().forEach(m => MENU_INDEX[m.name] = m)
const venue = { stageBenchmarkMin: { seated:9, starters:22, mains:38, dessert:18, check:12 } };
const swBench = { medianDecideMin: 6 };            // venue median menu-open → first submit
const staff  = [ {name:'James', role:'server', zone:'indoor',  onShift:true},
                 {name:'Maria', role:'server', zone:'indoor',  onShift:true},
                 {name:'Aisha', role:'server', zone:'patio',   onShift:true},
                 {name:'Tom',   role:'server', zone:'outdoor', onShift:true},
                 {name:'Nadia', role:'manager',zone:'all',     onShift:true} ];
const actionEvents = [];               // append-only learning stream (§8.3) — replaces the loose actionLog
let signalStats = {}, serverStats = {}, shiftStats = { medianActionLatencyMin:0, floorDoneRate:0, openActionCount:0 };
// state additions:  currentStaffName:'James',  actionOverflowOpen:null
```

**MENU changes (two lines, unblocks 4 signals):**
```js
// tags vocabulary MUST match ALLERGY_LABEL, lowercased
starters: [ {name:'Burrata', tags:['dairy'], flavourTags:['cream','fresh'], pairsWith:['Sauvignon Blanc']},
            {name:'Truffle Arancini', tags:['gluten','dairy','egg'], flavourTags:['truffle','cream'], pairsWith:['Barolo 2018']}, … ]
mains:    [ {name:'Wagyu Ribeye', tags:[], flavourTags:['red-meat','rich'], pairsWith:['Barolo 2018']},
            {name:'Sea Bass', tags:[], flavourTags:['citrus','light'], pairsWith:['Sauvignon Blanc']}, … ]
// add tags: 'nuts','shellfish','egg','soy','pork','alcohol'
```
Plus: **canonicalise the seed order lines.** `'GF Risotto'`, `'Insalata (GF)'`, `'Pizza Diavola'`, `'Vitello Tonnato'`, `'Margherita'`, `'Caesar Salad'`, `'Beef Carpaccio'`, `'Crudo'`, `'Tagliata'` are not in `MENU`, so `MENU_INDEX[i.name]` is undefined and `unsafe-plate` silently misses. Either rename them to MENU names or add `menuId` and index on that. **This is the single highest-risk data gap in the whole spec** — the UI prints a confident "gluten-safe" stamp over dishes it cannot check.

---

## 3. PRIORITY & RANKING ALGORITHM

Replaces the sort in `tableSignals` (4131–4138). Deterministic — the same table state must produce the same order on every render, or cards flicker.

### 3.1 Collection

```js
function tableSignals(t, pol) {
  if (!t || t.status === 'empty') return EMPTY_SIG;
  const c = signalContext(t, pol);

  const evaluate = list => list.filter(s => { try { return !!s.when(c); } catch(e){ return false; } })
    .filter(s => s.dataToday !== false || DEV_SHOW_ROADMAP)      // unmeasurable never ranks
    .map(s => ({ ...instanceOf(t, s, c), rankScore: 0 }));

  const waiterAll = evaluate(SIGNALS.filter(s => s.audience === 'waiter'));
  c._attnCount = waiterAll.filter(s => s.tier === 'attention' && !s.actioned).length;   // mgr-multiple reads this
  const managerAll = evaluate(SIGNALS.filter(s => s.audience === 'manager'));
  …
}
```

A signal whose trigger reads a `null` clock (`c.serveGap`, `c.attnAge`, `c.bill`) **must not fire** — `null >= 18` is false in JS, which is the behaviour we want, but never write `(c.serveGap||0) >= 18`: that turns "unmeasurable" into "fine".

### 3.2 Score

```js
const TIER_BASE = { attention:700, opportunity:400, context:100 };
const SEV_BOOST = { red:120, amber:50, neutral:0 };

function rankScore(s, c) {
  let v = TIER_BASE[s.tier] + (s.score || 0) + SEV_BOOST[s.severity];

  // urgency — only for signals that declare a threshold + rate, and only from real clocks
  if (s.urgency) v += Math.min(60, Math.max(0, s.urgency(c)) * (s.rate || 1));

  // guestWeight, capped +30
  let g = 0;
  const p = c.profiles;
  if (p.some(x => x.visits >= 5 || ['VIP','Network Regular'].includes(x.vipTier))) g += 18;
  if (c.churn) g += 12;
  if (c.occasion) g += 10;
  if (p.some(x => avgRating(x) && avgRating(x) <= 3.5)) g += 8;
  v += Math.min(30, g);

  // stageFit — an action that cannot be executed at this stage is not an action
  if (s.stages && !s.stages.includes(c.stage)) v -= 80;

  // fatigue — this server, this shift
  const f = (serverStats[c.t.server] || {}).dismissed || {};
  v -= Math.min(45, 15 * (f[s.id] || 0));
  if ((signalStats[s.id] || {}).dismissRate14d > 0.6) v -= 30;

  // severity ceiling: a red-ineligible signal can never out-rank on colour alone
  if (s.severity === 'red' && !RED_ALLOWED.has(s.id)) v -= SEV_BOOST.red;
  return v;
}
```

### 3.3 Sort — strict tie-break order

```js
const byRank = (a,b) =>
     (SIG_TIER_RANK[a.tier] - SIG_TIER_RANK[b.tier])        // 1. attention > opportunity > context, always
  || (SEV_RANK[a.severity] - SEV_RANK[b.severity])          // 2. red > amber > neutral
  || (Number(b.risk||false) - Number(a.risk||false))        // 3. safety before revenue
  || (b.rankScore - a.rankScore)                            // 4. score
  || ((a.effortSec||60) - (b.effortSec||60))                // 5. the one-tap thing beats the go-find-the-chef thing
  || (a.detectedAtMin - b.detectedAtMin)                    // 6. FIFO — nothing starves
  || (a.id < b.id ? -1 : 1);                                // 7. stable
```

### 3.4 Caps

```js
const ranked  = attention.concat(opportunity).sort(byRank);
const primary = ranked.slice(0, MAX_PRIMARY);   // MAX_PRIMARY = 2, already in the file
const overflow= ranked.slice(MAX_PRIMARY);
```

- **Hard max 2 primary action cards per table.** Slot 1 = `.act-card` + `.lead` (champagne emphasis). Slot 2 = plain `.act-card`. Never two leads.
- **Opportunity never displaces attention.** It reaches a slot only when `attention.length < 2`.
- **Ranks 3..N collapse into exactly one row** — the existing `.act-more` button: `+${n} more suggestions for this table` → `toggleActMore(t.id)`. Expanded shows at most **5**, each as a 34px compact row with its own one-tap button. `state.actExpanded[t.id]` **resets on modal close** (add to `closeModal`). It never auto-expands, and collapsed items are **not marked `seen`**.
- **Context is never a card.** `tier:'context'` renders as `.ctx-chip` in `renderContextStrip` (max 5 chips) or as `.ai-bullet` rows inside the collapsed guest section — max 4 + the always-last `users` bullet from `buildServiceSummary`.
- **Floor tile: zero cards, zero buttons.** One health dot, one count, one action line (§6).
- **Novelty cap:** `t.cardsSurfacedCount >= 6` for a `seatCycle` ⇒ only `tier:'attention'` breaks through. Stops a two-hour table becoming a card treadmill.
- **Recompute triggers:** modal open, `simulateOrder`, `advanceCourse`/`advanceDrink`, status change, done/dismiss, `confirmSeat`, `doMoveTable`, and a **60s tick** from `updateClock` (5452) that also increments `nowMin`. **Not** on every `renderFloor()` beyond the tile read.

### 3.5 Colour discipline (an allowlist, not a judgement call)

```js
const RED_ALLOWED = new Set([
  'allergy-brief','unsafe-plate','basket-allergy-conflict',      // 1. unconfirmed live dietary risk
  'guest-request','check-waiting',                               // 2. explicit guest help signal
  'course-overdue','course-gap','seated-no-order',               // 3. unattended wait past a hard threshold
  'action-sla-breach','mgr-flag-unattended','mgr-request-unanswered', // 4. breached SLA on a red action
  'order-submit-failed','payment-failed',                        // 5. stuck check / payment
  'open-issue-tonight','prev-service-issue','mgr-recovery','mgr-recovery-repeat','mgr-multiple','mgr-overdue-vip'
]);
```

- **Max ONE red element visible per table view.** A second red-eligible signal renders **amber** (`s.severity = 'amber'` at instance level, catalog untouched) and raises `mgr-multiple` instead of stacking.
- **amber** = attention without harm: pacing gaps, dry glasses, aging tickets, an unmarked occasion at dessert, an action nobody has looked at.
- **neutral** = every opportunity, every context row.
- **Champagne (`--accent`) is hierarchy, never urgency** — it marks the lead slot and the primary button (`.act-do`). **Green is confirmation only** — the `.act-done-row` check glyph and success toasts. Never a severity.
- **Red budget guard:** if reds would exceed **30% of occupied tables**, the ranker demotes the lowest-scoring reds to amber and fires `mgr-red-budget`. A sea of red is a ranking bug; the system self-limits rather than shouting.
- Waiter-facing context rows get **no task verb** — `Got it` / `Noted`. Only manager-facing context (a browsable view) gets real verbs.

---

## 4. EXPERIENCE HEALTH

Health is **never a score**. It is the worst severity among the signals actually firing on this table, and its reason string is a real signal's own short title. Replaces `tableHealth` (4151).

```js
function tableHealth(t, attention, opportunity, c) {
  if (!t || t.status === 'empty') return { level:'none', reason:'', count:0 };

  // ONLY waiter-audience attention signals colour a table. Manager signals and
  // historical recovery flags never make a table red — they surface as a muted
  // "extra care" marker, so red stays rare and means "measurable live failure".
  const live = attention.filter(s => s.audience === 'waiter' && !s.historical && !s.actioned);
  const care = attention.filter(s => s.historical && !s.actioned);
  const openCount = attention.filter(s=>!s.actioned).length + opportunity.filter(s=>!s.actioned).length;

  const top = live[0];                          // already sorted by byRank
  if (top && top.severity === 'red')   return { level:'red',   reason: reasonShort(top, c), count:openCount, care:care.length };
  if (top && top.severity === 'amber') return { level:'amber', reason: reasonShort(top, c), count:openCount, care:care.length };

  if (t.status === 'paid')      return { level:'green', reason:'Bill paid · ready to turn',      count:openCount, care:care.length };
  if (c.stage === 'check')      return { level:'green', reason:'Wrapping up · nothing pending',  count:openCount, care:care.length };
  if (openCount)                return { level:'green', reason:'On pace · '+openCount+' suggestion'+(openCount>1?'s':''), count:openCount, care:care.length };
  return                               { level:'green', reason:'On pace · nothing waiting',     count:0,         care:care.length };
}
```

`reasonShort` is a **new required property on every attention signal** — ≤ 34 characters, measurable, no adjectives. It is what the tile and the `.health-pill` show:

```js
reasonShort: c => `Asked for you ${c.sw.requestMin}m ago`   // guest-request
reasonShort: c => `Seated ${c.mins}m, no order`             // seated-no-order
reasonShort: c => `Nothing served ${c.serveGap}m`           // course-gap
reasonShort: c => `Round poured ${c.t.drinkAgeMin}m ago`    // drinks-low
reasonShort: c => `Waiting on the check ${n}m`              // check-waiting
reasonShort: c => `Kitchen not briefed · ${c.allergies[0]}` // allergy-brief
reasonShort: c => `${c.occType} unmarked at dessert`        // occasion-last-chance
```

**Rules:**
1. Every level is traceable to one firing signal id — `health.reason` is never generated prose.
2. `level:'green'` with `count > 0` renders the tile dot in **`.tile-health.neutral`** (champagne), not green — there are things to do, they just aren't failures. This behaviour already exists at 3331 and is correct; keep it.
3. Service **recovery never colours the tile**. `health.care > 0` adds a muted `.tile-care` marker (see §6).
4. A table with no measurable clocks (`serveGap === null` etc.) can still be green — we do not invent amber out of missing data.

---

## 5. TABLE VIEW INFORMATION ARCHITECTURE

Required order: **NEEDS YOUR ATTENTION → OPPORTUNITY → GUEST (collapsed) → ORDER → SUGGEST NEXT.** The current `openTableModal` (3691) puts the full `.party-header` *above* the actions — that is the one structural change needed.

### 5.1 Table with actions — `.modal-body` children in order

| # | block | classes | notes |
|---|---|---|---|
| 1 | Health strip | `.health-pill ${level}` in `.modal-head` (**already there**, 3724) + new `.hcount` | Header carries identity + health so the body can open on actions. Sub-line becomes `Server ${server} · seated ${mins}m · ${guestName}` |
| 2 | **NEEDS YOUR ATTENTION** | `.sec-head.urgent` + `.act-card.red\|.amber` (+ `.lead` on slot 1) | max 2 total across §2+§3 combined |
| 3 | **OPPORTUNITY** | `.sec-head` + `.act-card.neutral` | only if fewer than 2 attention cards |
| 4 | Overflow | `.act-more` | `+N more suggestions for this table` |
| 5 | Done strip | `.sec-head` + `.act-done-row` ×n | quiet; the waiter must see their own round |
| 6 | **GUEST** (collapsed) | `.sec-head` + `.ctx-strip > .ctx-chip` + **new** `.party-header.slim` + `.disclose` / `.disclose-body` > `.ai-sum-text`, `.kv-grid`, `.rem-btn` | context chips are the only always-visible guest facts |
| 7 | **ORDER** | `.sec-head` + `.tdetail > .tcol` ("On the table") | `renderJourney` stays behind `pol.courseJourney` |
| 8 | **SUGGEST NEXT** | `.tcol.plan > #suggest-body` + `.col-action` Shuffle | unchanged `renderSuggestBody(plan)` |
| 9 | Footer | `.actions-row` + `.demo-btn` + `.btn` | unchanged |

**Class reuse vs new:**

*Reuse as-is:* `.sec-head(.urgent)`, `.act-card` + severity modifiers, `.act-title`, `.act-why`, `.act-do`, `.act-done-row`, `.act-more`, `.act-clear`, `.ctx-strip`/`.ctx-chip`, `.disclose`/`.disclose-body`, `.kv-grid`/`.kv`, `.rem-btn`, `.health-pill`, `.tcol`/`.tcol-head`/`.tcol-body`, `.tdetail`, `.plan-card` family, `.bal-cell`, `.actions-row`, `.live-tag`.

*New CSS needed (≈40 lines):*
```css
.act-card.lead { background: linear-gradient(180deg, rgba(216,195,154,0.07), transparent 55%), var(--surface-2);
                 border-color:#3a3324; box-shadow:0 10px 30px -16px rgba(216,195,154,0.35); }
.act-card.compact { padding:9px 12px; margin-bottom:6px; display:flex; align-items:center; gap:10px; }
.act-card.compact .act-why { display:none; }
.act-card.compact .act-do { width:auto; padding:6px 12px; font-size:11.5px; margin-left:auto; }
.act-window { font-size:10.5px; text-transform:uppercase; letter-spacing:.7px; color:var(--text-faint);
              border:1px solid var(--border-strong); border-radius:999px; padding:2px 8px; margin-left:8px; }
.act-dismiss { margin-left:auto; font-size:11px; color:var(--text-faint); }
.party-header.slim { padding:12px 14px; gap:12px; }
.party-header.slim .party-name { font-size:20px; }
.party-header.slim .party-avatar { width:32px; height:32px; font-size:12px; }
.health-pill .hcount { font-family:var(--font-mono); font-size:10px; margin-left:6px; opacity:.8; }
.care-note { display:flex; gap:8px; align-items:center; font-size:12px; color:var(--text-dim);
             background:var(--surface-2); border:1px dashed var(--border-strong);
             border-radius:var(--r-sm); padding:8px 12px; margin-bottom:12px; }
```
**Glass obligation:** add `.act-card, .ctx-chip, .mgr-card` to the `body.glass` override selector list (~2196 in the old numbering / search `body.glass`), or these cards keep a solid background under the frosted theme.

### 5.2 Healthy table with no actions — the minimal variant

Header pill reads `On pace · nothing waiting` (green). Body:

1. `.act-clear` — *"Experience looks healthy — nothing needs you right now."* (**already implemented**, 4227)
2. `.ctx-strip` — the 1–3 facts that would change service anyway (allergy, occasion, "avoid spicy"). If empty, omit the strip entirely.
3. **GUEST**, collapsed. Nothing auto-opens.
4. **ORDER** + **SUGGEST NEXT** — unchanged; on a healthy table these are the point of the screen.

No section headers for empty sections. No "0 items" states. A healthy table's modal should be visibly shorter than a flagged one — that difference *is* the signal.

---

## 6. FLOOR VIEW

One health indicator + one action count + at most one safety glyph + one action line. The current `renderTile` (3329–3352) already does this; the changes are the `care` marker, the `reasonShort` footer, and a red-only footer rule.

```html
<div class="table-tile ${t.status}" data-id="${t.id}" onclick="openTableModal(${t.id})">
  <div class="tile-top">
    <div class="tile-num">${t.id}</div>
    <div class="tile-ind">
      ${allergyGlyph}                                  <!-- .tile-glyphs > .tile-glyph.red — allergy ONLY, max 1 -->
      ${care ? `<span class="tile-care" title="Extra care">${ic('heart-handshake','xs')}</span>` : ''}
      <span class="tile-health ${hLevel}">
        <span class="hdot"></span>${count ? `<span class="hcount">${count}</span>` : ''}
      </span>
    </div>
  </div>
  <div class="tile-party">${partyName}</div>
  <div class="tile-pax">${ic('users','xs')} ${pax} pax${ratingHtml}</div>
  <div class="tile-bottom ${timeClass}">
    ${top && top.severity !== 'neutral'
      ? `<span class="tile-action ${top.severity}">${ic(top.icon,'xs')} ${health.reason}</span>`
      : `<span>${time}m · ${courseWord}</span>`}
    <span class="tile-server">${t.server || ''}</span>
  </div>
</div>
```

```js
const sig    = tableSignals(t, pol);
const count  = sig.attention.filter(s=>!s.actioned).length + sig.opportunity.filter(s=>!s.actioned).length;
const hLevel = sig.health.level === 'green' && count ? 'neutral' : sig.health.level;
const care   = sig.health.care > 0;
const top    = sig.primary[0];
```

**Rules the tile must obey:**
- `hdot` colour is the **only** status carrier that changes with health; the existing `::before` status bar and the `needsEye`/`needsEyeRed` pulses stay driven by `t.status` (2323–2350) — do not double-encode.
- `.hcount` is the count of **open actions**, not of signals detected. Suppressed at 0.
- Footer shows `health.reason` **only when the top signal is red or amber**. A neutral opportunity does not earn the footer — the count already says there is something inside.
- Max **one** glyph, and only `glyphCat === 'allergy'` (already enforced at 3323). Occasion, VIP and attention glyphs live inside the modal.
- Zero buttons on a tile. Ever.
- New CSS: `.tile-ind{display:flex;align-items:center;gap:8px}` and `.tile-care{color:var(--text-faint);opacity:.75}` (12px icon).

---

## 7. MANAGER VIEW

Extends `renderManager()` (5113). It is a **queue, not a floor plan**: no zone tabs, no search, no grid. If a manager has to filter, we failed.

**Header** — `.floor-header` with `.floor-title.serif` = *"Your next ten minutes"* and three `.stat` blocks: `Needs you` · `Worth your time` · `Quiet` (live tables no manager has been near). Subtitle: `${n} tables need you personally · ${covers} covers on the floor`.

**Sections, top to bottom** (`.mgr-group` + `.sec-head` each):

| # | section | contents | ordering | cap | colour |
|---|---|---|---|---|---|
| 1 | `${ic('circle-alert')} Needs you now` | manager `tier:'attention'` | `byRank` | **5** cards, then `+N more urgent` | max **2** red; a third collapses to `+1 more urgent` |
| 2 | `${ic('crown')} Worth your time` | manager `tier:'opportunity'` | `byRank` | 4 | neutral/champagne only, each with an `.act-window` chip: `before dessert` / `before they pay` / `first 15 min` |
| 3 | `${ic('door-open')} At the door` | `reservationSignals()` for arrivals ≤30 min | **`r.minutesUntil` asc** — the clock owns this section, not priority | 4 | amber max |
| 4 | `${ic('users-round')} Sections` | one `.mgr-srv` row per `staff` server | flagged desc | all | neutral |
| 5 | `${ic('moon')} Tonight` | manager `tier:'context'`, **collapsed** via `.disclose` | score desc | all | no red |

**Row-level rules:** ONE card per table in sections 1–2 — the table's highest-ranked manager signal wins, the rest collapse into `+${n} more on this table` (tap expands). A table in section 1 never repeats in section 2.

**Markup** — reuse `.mgr-card`/`.mgr-t`/`.mt-num`/`.mt-lbl`/`.mgr-body`/`.mgr-guest`/`.mgr-why`/`.mgr-tags`/`.mgr-act`/`.act-do`/`.mgr-empty`/`.me-ic` (all already styled). New: the section board row and the window chip.

```html
<!-- Section 4 row -->
<div class="mgr-srv">
  <div class="ms-name">${s.name}<span class="ms-zone">${s.zone}</span></div>
  <div class="ms-nums">
    <span class="ms-n">${live} live</span>
    <span class="ms-n ${flagged?'warn':''}">${flagged} flagged</span>
    <span class="ms-n">oldest untouched ${oldestTouch}m</span>
    <span class="ms-n">${openActions} open</span>
  </div>
  <button class="btn ghost" onclick="openMoveTable(${busiestTableId})">${ic('arrow-right-left','sm')} Move a table</button>
</div>
```
```css
.mgr-srv{display:flex;align-items:center;gap:16px;padding:12px 16px;background:var(--surface-2);
         border:1px solid var(--border);border-radius:var(--r-sm);margin-bottom:8px;}
.mgr-srv .ms-name{font-weight:600;font-size:14px;min-width:150px;}
.mgr-srv .ms-zone{color:var(--text-faint);font-size:11px;margin-left:8px;text-transform:uppercase;letter-spacing:.6px;}
.mgr-srv .ms-nums{display:flex;gap:16px;flex:1;font-size:12px;color:var(--text-dim);font-family:var(--font-mono);}
.mgr-srv .ms-n.warn{color:var(--amber);}
```

**Empty state is a feature.** When sections 1–2 are both empty:
```html
<div class="mgr-empty"><span class="me-ic">${ic('check')}</span>
  <span>Nothing needs you. Tables ${quiet.join(' and ')} haven't seen you all service.</span></div>
```
Idle time is the product's real opportunity — never render a blank section.

**Done behaviour:** `markAction(id,'manager')` collapses the card into a quiet strip at the bottom of its section — `Table 8 · visited 4m ago` (`.act-done-row`) — sets `t.mgrVisitMin = 0`, and suppresses that signal for the rest of the `seatCycle`.

---

## 8. ACTION LIFECYCLE + REMEMBER THIS

### 8.1 The instance model

Signals stay **pure functions**. The stateful object lives on the table so it moves with `doMoveTable`. This replaces the flat `actionState{}` (3879) — keep `isActioned()` as a thin wrapper so no existing call site breaks.

```js
t.actions = [{
  key: `${t.id}:${signalId}:${t.seatCycle}`,   // scoped to the PARTY, not the table
  signalId, tier, severity, risk, effortSec,
  status: 'detected',            // detected|seen|actioned|done|dismissed|expired
  detectedAtMin, seenAtMin, actionedAtMin, doneAtMin,
  dismissCount: 0, snoozeUntilMin: 0,
  recurring, rearmMin,           // 'drinks-low' re-arms after 20; 'allergy-brief' never does
  verifiable, triggerStillTrue,
  rank                           // slot at the moment it was surfaced (logged, not persisted UI state)
}];
function isActioned(tableId, id) {            // unchanged signature
  const t = tables.find(x => x.id === tableId);
  return !!t?.actions?.some(a => a.signalId === id && a.seatCycle === t.seatCycle && ['actioned','done'].includes(a.status));
}
```

**States:** `detected` (scored, never visible) → `seen` (**only** when `openTableModal` ran AND the card sits in slot 1 or 2 — items behind `+N more` are *not* seen; this is the only honest denominator for a done-rate; also sets `t.lastViewedAtMin = nowMin`) → `actioned` (the tap; optimistic and instant — `.act-do` morphs in place to `✓ ${doneLabel}` for 900ms via the existing `.act-do-done` span; no spinner, no confirm dialog) → `done`. Plus `dismissed` (`snoozeUntilMin = nowMin + (tier==='attention'?15:30)`, `dismissCount++`; returns only if severity escalates a step) and `expired` (trigger went false with no staff action — fades silently, **no toast**; a high expiry rate means we were slow, not that staff ignored us).

### 8.2 What collapses on done — and what must mutate

1. The card **collapses** (height animates to the 34px `.act-done-row`), it does not vanish. The waiter must see their own tap land. `.act-card.clearing` + `actOut` already exist (887).
2. **Every done writes a real field**, or the next render exposes the lie:

| signal | done-effect |
|---|---|
| `allergy-brief` | `t.allergyBriefedAt = nowMin` |
| `occasion-setup`, `occasion-last-chance`, `mgr-occasion` | `t.occasionMarkedAt = nowMin` |
| `drinks-low`, `bottle-not-poured`, `drinks-not-poured` | `t.drinkAgeMin = 0` (`simulateOrder` already does exactly this — reuse it) |
| `check-waiting` | `t.progress[3] = true` |
| `guest-request`, `mgr-request-unanswered` | `delete t.sw.requestPending; t.sw.requestMin = 0` |
| `open-issue-tonight` | `t.issue.resolved = true` |
| any manager card | `t.mgrVisitMin = 0` |
| every done | `t.lastStaffTouchMin = 0` |

3. Row holds for **6s with an inline Undo** (`.act-undo` → `undoAction`), then removes itself. `toast('${doneLabel} · table ${t.id}','success')` — operational text only, **never "AI learned from this"**.
4. After the hold, the ranker re-runs and promotes **exactly one** card from overflow into the freed slot with `planIn`. One card animates; the list never reshuffles wholesale — the waiter's eye must not lose its place.
5. `t.actions` is **never purged mid-cycle** — the shift-level and post-meal signals (`mgr-table-finished-untouched`, `action-done-unverified`) read it. Cleared only by `confirmSeat` (new `seatCycle`).

### 8.3 Emitted event — one append-only record per transition

```js
actionEvents.push({
  eventId, ts: Date.now(), nowMin, venueId:'bella-notte', shiftId, serverId: t.server,
  tableId: t.id, seatCycle: t.seatCycle, zone: t.zone,
  signalId, signalTier, severity, priorityScore, rankSlot,
  cardsVisible, collapsedCount,                       // was it competing or alone?
  triggerSnapshot: pick(c, SIGNALS_BY_ID[signalId].fieldsUsed),   // ONLY that signal's declared fields
  guestRefs: t.party, guestVisits, guestAvgRating, guestVipTier,
  recommendedAction: { key: signalId, label: actionLabel },
  staffAction: 'seen'|'done'|'dismissed'|'undone'|'expired',
  dismissReason: null|'already-done'|'not-true'|'not-now',
  seenLatencyMs, dwellMs, latencyMs,                  // detected→seen, seen→done, detected→done
  outcome: null
});
```

`outcome` is back-filled at check close from systems we already own (POS + payment + feedback):
`{ tableRating, ratingDeltaVsGuestAvg, checkTotal, checkDeltaVsPredicted, dessertAttach, upsellAccepted, returnedWithin90d }`.

The loop is **aggregate, never per-event causal**. Per `signalId` we compare done vs dismissed vs expired cohorts on rating and attach rate, split by venue and by server. Three permitted uses: (a) recalibrate `score` weights; (b) retire noise — >60% dismissal over 20+ surfaces demotes one tier, then retires (`signal-noise-review`); (c) coach (`mgr-server-coaching-gap`). We do **not** invent new signals from the loop, and we **never** show a staff member a model confidence number.

### 8.4 Remember This — three taps, typing optional

Entry points: the existing `.rem-btn` in the guest section (4285), a `+ Remember` button in `.modal-head`, and long-press on any `.party-avatar`. The 6-category `#remember-modal` **already exists** (HTML 2834–2866, `openRemember`/`pickRemCat`/`saveRemember` 4300–4336). The upgrade is per-category minimal inputs and structured landing.

1. Tap `+ Remember` → `.modal.small`, six `.rem-cat` chips. **No text field visible yet.**
2. Tap a category → the smallest possible input for that category renders **already focused**:

| category | input | typing |
|---|---|---|
| **Preference** | one line, 40 chars + `Likes / Avoids` toggle + **4 one-tap suggestions built from what is literally on the table now** (`t.items[].name`, `t.drinks[].name`) | usually none |
| **Dietary** | chips from `ALLERGY_LABEL` (nuts/shellfish/dairy/gluten/vegetarian/vegan/halal) + `Allergy / Preference` severity toggle | none |
| **Occasion** | chips from `OCCASION_LABEL` + `this visit / every year`; "every year" = two spinners (month, day) | none |
| **Seating** | chips: *this table* (prefilled `Table ${t.id}`), quiet corner, patio, banquette, away from kitchen, high top | none |
| **Conversation** | one line, 60 chars, labelled *for the next host* | one line |
| **Other** | one line, 60 chars | one line |

3. `Save` → sheet closes, `toast('Remembered · on ${first}\'s profile','success')`, the new `.ctx-chip` **flashes champagne for 1.5s** inside the guest section so the waiter sees exactly where it landed, `t.memoryAddedCount++`.

**Canonical store + mirrored writes** (mirrors are what make taught memory useful *immediately*, using only fields the app already consumes):

```js
guests[gid].memory.push({ id, cat:'preference'|'dietary'|'occasion'|'seating'|'conversation'|'other',
  value, polarity:'like'|'avoid'|null, severity:'allergy'|'preference'|null,
  recurring:{month,day}|null, scope:'guest'|'visit',
  tableId, seatCycle, byServer: state.currentStaffName, ts: Date.now(),
  surfacedCount:0, confirmedCount:0, lastSurfacedVisit:null, source:'staff' });
```

| category | mirror → consumed by |
|---|---|
| dietary + `allergy` | Title-Case label → `guests[gid].allergies` **and** `t.allergies` → `allergyKeysFor`, `buildServiceSummary` lead, red `ALLERGY_GLYPH` tile glyph on the next render |
| dietary + `preference`, or preference + `avoid` | `guests[gid].dislikes` → the amber dislikes bullet + `.ctx-chip` |
| preference + `like` | `favFood` if it matches a `MENU` name, else `favDrinks` if it matches a `t.drinks[].name`, else memory-only → improves `suggestedMain`/`upsell`/the `usual` tag without inventing anything |
| occasion · this visit | `t.occasion` + `OCCASION_GLYPH` |
| occasion · every year | `guests[gid].occasionCalendar[]` → `res-recurring-occasion` |
| seating | `guests[gid].seatingPref` (prose) + `seatPref.zone` when a zone chip was used → seat-match card and reservation flow, **never the floor view** |
| conversation / other | memory-only |

**Resurfacing:** `scope:'guest'` entries enter the next visit's context set as `tier:'context'` — **except** dietary/allergy, which enters as `attention`/red, and occasion, which enters as `opportunity`/neutral. Every surfaced row carries a human attribution line — `Remembered by Maria · 12 Mar`. **No AI framing anywhere**; ownership by the floor team is what keeps them teaching it. `surfacedCount++` per visit shown; acting on it → `confirmedCount++`. `confirmedCount >= 2` promotes a preference into the suggestion engine's `usual` tag. `surfacedCount >= 3 && confirmedCount === 0` raises the one-tap `memory-stale-confirm` — **in the briefing, never on the floor**. A contradicting memory never silently overwrites: both stack, newest first, with "changed from X" in the briefing (`memory-conflict`). Every save emits an `actionEvent` with `signalId:'remember-this'`, so we can measure whether taught memory actually raises repeat-visit ratings — **the only claim the learning loop is allowed to make**.

---

## 9. IMPLEMENTATION ORDER

Ten steps, each independently shippable and independently revertable. Every step names the exact functions it touches. **Nothing before step 6 changes existing markup**, so the guided tour selectors (`.table-tile[data-id="1"]`, `#table-modal .ai-summary`, `#table-modal .tcol.plan`, `#table-modal .party-header`, `#seat-cta`, …) keep resolving.

**Step 1 — Clock and index (pure additions, zero risk).**
Add `nowMin`, `MENU_INDEX`, `venue`, `swBench`, `staff`, `actionEvents`, `signalStats`/`serverStats`/`shiftStats` near `state` (3123). Add `state.currentStaffName='James'`, `state.actionOverflowOpen=null`. Modify **`updateClock()` (5452)** to also `nowMin++` and, every 60s, call `renderFloor()` when `state.view==='floor'`. Nothing reads these yet.

**Step 2 — MENU truth (unblocks 4 red signals and removes a real safety lie).**
Modify **`MENU` (3810)**: add `'nuts','shellfish','egg','soy'` tags, plus `flavourTags` and `pairsWith`. **Canonicalise the seed order-line names** in `tables` (3054) so `MENU_INDEX[i.name]` resolves — or add `menuId` and index on it. Verify `allergyKeysFor`/`isSafe`/`buildServicePlan`'s `allergyLabel` still produce the same strings for tables 4 and 18 before moving on.

**Step 3 — Extend `signalContext` (3893).** Add every derived field from §1.0. Existing 22 signals keep reading `c.sw`, `c.mins`, `c.allergies` unchanged — this step must be a strict superset. Run the app; the floor and modal must look **identical**.

**Step 4 — Seed the new fields.** `guests` (2935): `daysSinceLastVisit`, `allergySeverity` on g1/g4/g8, `upsellAffinity` on all, `seatPref` on g3/g8, `lastFeedback` on g7, `serviceIssues` on g7, `orderHistoryItems` on g8, `memory` on g3. `tables` (3054): extend `sw` per §2.1, add `seatCycle:1`, `lastServeMin`, `lastOrderMin`, `lastStaffTouchMin`, `attentionMin`, `stageEnteredMin`, `billTotal` on occupied tables; add `issue` to table 12 and `recovery` to table 5 so the recovery path is demoable.

**Step 5 — Grow the catalog (`SIGNALS`, 3922).** Add signals in **priority order, in batches of ~10**, checking the floor after each batch. Every new entry must carry `score`, `reasonShort`, `risk`, `effortSec`, `stages`, `fieldsUsed`, `dataToday`. Add `dataToday:false` entries too — they are gated out by `evaluate` in §3.1 and become the roadmap in code. **Do not touch the 22 existing entries' ids** — the tour, `markAction` and `actionLog` all key off them.

**Step 6 — Ranker + colour discipline.** Modify **`tableSignals` (4111)**: `rankScore`, `byRank`, the `RED_ALLOWED` demotion, the one-red-per-table rule, the novelty cap. Modify **`tableHealth` (4151)** for `reasonShort` / `care` / `count`. Add `redShare()`, `serverLoad()`, `zoneLoad()`, `maxOpenBill()` helpers next to `paxOf` (3860). Nothing renders differently except which two cards win — verify tables 1, 4, 5, 12, 15 by hand.

**Step 7 — Floor tile.** Modify **`renderTile` (3289)**: `.tile-ind` wrapper, `.tile-care`, `health.reason` footer, red/amber-only footer rule. Add the three new CSS rules. `renderCardVariant` (3380) and `renderMapTable` (3546) get the same `.tile-health` treatment or explicitly none — do not leave the map showing stale status-only colour.

**Step 8 — Table modal IA.** Modify **`openTableModal` (3691)**: move `.party-header` into `renderGuestSection` as `.party-header.slim`, put `renderTableActions` first, add the `${ic('clipboard-list')} Order` head before `.tdetail`. Modify **`renderTableActions` (4200)** for `.lead`, `.act-window`, `.act-dismiss`, the compact overflow rows, and the `.care-note`. **Then update `coachSteps` (search `coachSteps`)**: the step whose `sel` is `#table-modal .party-header` must become `#table-modal .ctx-strip` or `#table-modal .disclose`, and the `.ai-summary` step must point at the first `.act-card` — otherwise `waitForCoachEl` times out and the tour falls back to `positionCenter`. Add `delete state.actExpanded[id]` to **`closeModal` (5273)**.

**Step 9 — Lifecycle + Remember This.** Migrate `actionState` → `t.actions` behind the existing `isActioned` signature. Modify **`markAction` (4163)** for the done-effects table in §8.2, the 6s hold, the single-card promotion, and `actionEvents.push`. Modify **`undoAction` (4173)** to emit `staffAction:'undone'`. Add `dismissAction(tableId, id, reason)`. Modify **`saveRemember` (4321)** to write the structured `guests[gid].memory` entry plus mirrors; extend `pickRemCat` (4315) with the per-category inputs. Modify **`confirmSeat` (4803)** to `t.seatCycle++`, clear `t.actions`, `t.allergyBriefedAt`, `t.occasionMarkedAt`, `t.memoryAddedCount`, `t.cardsSurfacedCount`. Modify **`doMoveTable` (4697)** with the extended transfer key list from §2.2.

**Step 10 — Manager view + door signals.** Modify **`renderManager` (5113)** into the five sections of §7. Add `reservationSignals(r)` and `RES_SIGNALS[]` next to `SIGNALS`, plus `r.minutesUntil` derived from `r.time` against the real clock (`updateClock`). Add `.mgr-srv` CSS. Finally add `.act-card, .ctx-chip, .mgr-card, .mgr-srv` to the `body.glass` override list, and re-check the ≤900px breakpoint — `.act-do` is full-width there and the two-card cap matters most on a tablet held in one hand.

**Sequencing rationale:** data truth (1–4) before logic (5–6) before pixels (7–8) before state machines (9) before the new surface (10). Steps 1–5 are additive and cannot regress the running prototype; the first behaviour change a user can see is step 6, by which point every trigger it ranks is already reading real, seeded fields.