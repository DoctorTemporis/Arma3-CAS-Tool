# Roadmap — Arma 3 CAS / FAC Tracker

Working document for planned enhancements. Work top-down through **Build order**; each
feature below has enough detail to implement without the originating conversation.

---

## Project context (read first in a fresh session)

- **What it is:** a single self-contained `CAS-Tracker.html` — no server, no database, no
  install. Double-click to run in a browser. Auto-saves to `localStorage`, with JSON
  export/import.
- **Who uses it:** one GM ("sole operator"), live, while simultaneously running an Arma 3
  session for 10–20 players and voicing both the control agency and the pilots on the radio.
  **Cognitive load during play is the binding constraint** — features that reduce it beat
  features that add fidelity.
- **Setting:** Vietnam, S.O.G. Prairie Fire + Unsung Redux airframes.
- **Core fiction:** on-call *immediate* CAS — aircraft diverted from other tasking, so
  variable fuel/playtime and partial loads. Not pre-planned strikes.
- **Workflow:** FAC requests → control agency offers options → FAC picks → `EN ROUTE` →
  `AWAITING CHECK-IN` → `ON STATION` → strikes → departure.

**Repo/workflow:** private GitHub repo `DoctorTemporis/Arma3-CAS-Tool`, branch `main`.
Commit **and push after every change** (user preference). `gh` is not on PATH; full path:
`C:\Users\losti\AppData\Local\Microsoft\WinGet\Packages\GitHub.cli_Microsoft.Winget.Source_8wekyb3d8bbwe\bin\gh.exe`

**Released:** `v1` (initial), `v2` (ordnance accounting + per-aircraft employment).
**On `main`, unreleased:** multiple named loadout configurations per airframe.

---

## Build order

Ordered for delivery (effort/risk aware), which differs from raw magnitude of improvement.
Magnitude rank is noted per item.

1. **Event alerts** — *magnitude #3, effort S* ← start here
2. **Radio prompt cards** — *magnitude #1, effort M*
3. **AAA battle damage** — *magnitude #2, effort M*
4. **Target & BDA board** — *magnitude #4, effort M/L*
5. **Squadron roster / campaign** — *magnitude #5, effort L* (deferred)

Suggested: **1 + 2 = v3**, **3 = v4**, then reassess.

---

## 1. Time-critical event alerts  *(effort: S)*

Audio + unmistakable visual cues for: flight arriving overhead, check-in pending,
5/2/1-minute bingo warnings, winchester, station expiry.

**Pros**
- Best effort-to-value ratio on this list. Self-contained; no data-model change.
- Fixes the tool's biggest failure mode: the GM is watching the *game*, not the tracker, so
  silent countdowns get missed — and a missed bingo call breaks immersion.
- Value scales with simultaneous flights, which are common in this group's sessions.

**Cons**
- Audio competes with live voice comms — needs volume control and conservative defaults.
- Alert fatigue if over-specified.
- Browser autoplay policy: audio needs one user interaction to unlock.

**Recommendation:** build first. Default to *few* alerts (arrival, 2-min bingo, winchester),
with per-event toggles + volume in TUNING. Distinct tones per event type.

**Implementation notes**
- Generate tones via `AudioContext` oscillator — no asset files, preserves single-file rule.
- Hook the existing 1 s `setInterval` tick; fire on threshold *crossings*, not every tick
  (store a `lastAlert` marker per flight to avoid repeats).
- Visual: pulse/flash the flight card + document title change so it's visible when the
  browser is in the background.

---

## 2. Radio prompt cards  *(effort: M)*

**Decided scope: on-screen text for the GM to read aloud. NOT game integration.**
One click on a flight yields the exact words, generated from that flight's *live* state.

Real fighter check-in brief = number/type of aircraft, position/altitude, ordnance,
playtime, abort code — which maps almost exactly onto data already stored.

Example:
> "Monkey Mountain, NIGHTSKY FOUR TWO, flight of two Skyraiders, four by Mark eight-two,
> two by napalm, twenty mike-mike, playtime two five."

Call types: check-in, in-from-heading, winchester, bingo/RTB, departure.

**Pros**
- Biggest qualitative leap — turns a tracker into a co-GM that composes your lines.
- No new data required; pure presentation of existing state.
- Always reflects *current* ordnance/playtime, killing the "what did I say they had" problem.
- Consistent period phraseology even late in a long session.

**Cons**
- Templates are opinionated; user may want to reword heavily.
- Can feel robotic; every flight risks sounding identical without per-airframe flavour.
- **Only valuable if the friction is *composing* the calls.** If the GM already rattles them
  off fluently, this is low value — confirm before building.

**Recommendation:** build, with **templates editable in the Library** so phrasing is tunable.
Keep cards short — a glanceable card, not a paragraph. Phonetic-ready numbers (`SIX ZERO`),
plus a copy button.

**Explicitly rejected: live game/radio integration.** A sandboxed single HTML file cannot
read Arma state or inject audio into TFAR/ACRE2. Real integration needs an Arma extension
(`callExtension`) or file/socket bridge, a helper process, TTS, and audio routing into the
radio mod — i.e. a multi-component install with a background service, which breaks the
"no server, just double-click it" requirement. Do not revisit without the user changing that
constraint.

*Optional, not recommended:* browser speech synthesis could read calls aloud, but output goes
to speakers (not the radio net) without a virtual audio cable, and a synthetic voice likely
harms immersion more than it saves typing. Cheap to add as an off-by-default toggle if asked.

---

## 3. AAA battle damage & attrition  *(effort: M)*

Today the AAA toggle *only* nudges accuracy. Make it consequential: roll per pass while
engaged for **clean / hit-no-damage / damaged (accuracy penalty + early RTB) / heavy damage
(abort, crew status) / shot down**.

**Pros**
- Makes an existing toggle meaningful, and closes a real gap: `SHOT DOWN` is currently 100%
  manual GM fiat.
- Removes adjudication burden mid-session.
- Generates drama the FAC must react to; a damaged aircraft naturally feeds SAR/Sandy tasking.
- Reuses existing statuses (`rtb`, `aborted`, `shotdown`, `kia`) — little new structure.

**Cons**
- Randomness can wreck a planned set-piece — needs a GM veto.
- Easy to over-tune into a meat grinder; real loss rates were **low**.
- Rotary and fixed-wing need different curves (helos took far more small-arms damage).

**Recommendation:** build with a **"suggest, GM confirms" gate**, identical to the weapon
accuracy model — never auto-kill a flight. Default the *shootdown* slider very low and the
*damage* slider meaningfully higher; that asymmetry is the historically accurate shape.

**Calibration:** USAF sustained roughly **0.4 losses per 1,000 sorties** in SEA. Light AAA and
small arms (12.7/14.5 mm and smaller) downed more US aircraft than all other NVA systems
combined — so *hits* were common, outright losses rare.

---

## 4. Target & BDA board  *(effort: M/L)*

A lightweight target list (label, description, mark type — WP/smoke/talk-on), flights
*assigned* to targets, and an automatic **BDA rollup** per target and per flight
("3× on target, 2 short, 1 hung").

**This is NOT the formal 9-line form the user already declined.** The 9-line stays roleplayed
over the radio; this only holds targets and totals effects so BDA can be read back accurately.

**Pros**
- Closes the one genuinely unfinished loop: releases are logged but nothing accumulates into
  "what did this achieve."
- Directly supports multiple simultaneous flights — currently no structured way to know who is
  working which target.
- Makes the after-action log materially more useful for debriefs.

**Cons**
- New entity + real UI surface; heaviest of the "moderate" options.
- Risks creeping toward the formal-process tooling deliberately avoided.
- Effects adjudication remains a judgement call — the tool can total hits, not decide outcomes.

**Recommendation:** build minimal. Flat target list with a text label and auto-computed
hit/miss tally. **Resist** modelling target types and damage states — simulation rabbit hole,
poor returns for a live GM.

---

## 5. Squadron roster & campaign continuity  *(effort: L — deferred)*

Named pilots with persistent skill, sortie counts and accuracy history surviving across
sessions; permanent KIA/MIA; squadron attrition. `SANDY 31` becomes a recurring character.

**Pros**
- Uniquely suited to a *dedicated recurring group* — stakes compound across weeks.
- Losing a veteran the players know lands far harder than losing a random callsign.
- Storage, export/import and skill modelling already exist — mostly a layer over solved parts.
- Yields cross-session stats (squadron loss rate, best/worst shooters) for debriefs.

**Cons**
- Highest effort, slowest payoff — near-zero benefit in session one.
- Tension with the current "relaunch fresh each game" habit; needs disciplined export/import.
- Conflicts with the on-call divert fiction — random tasking means the same pilots *shouldn't*
  always appear; roster weighting must stay believable.

**Recommendation:** defer. Natural v4+ headline feature once live-session ergonomics are solid.
**Cheap down payment available now:** persist *pilot names only* on generated flights — most of
the flavour for a fraction of the work.

---

## Backlog (small items)

- **Guns per-aircraft.** Droppable ordnance honours the per-aircraft/per-flight toggle, but guns
  remain flight-level coarse (full/partial/winchester) in both modes. Make guns per-aircraft when
  the toggle is set to per-aircraft.
- **Control agency biases airframe selection.** Historically Sandy = A-1 RESCAP escort,
  Pedro = rescue helos. Currently the agency is a pure tracking tag. Would need the user's
  preferred airframe mix per station.
- **Editable callsign word pool in the UI**, persisted to `localStorage` (currently the
  `CALLSIGN_WORDS` array is edited in-file). User declined once on the mistaken basis that a UI
  version wouldn't persist — it would. Re-offer if editing the array becomes tedious.
- **Refine default loadout configs** from the user's own listings when they extract them; the
  current set is a researched first pass.

---

## Standing decisions & constraints (do not violate)

1. **Single self-contained HTML file.** No server, no database, no build step, no external
   assets/CDN. Everything inline.
2. **Suggest, never auto-apply.** The tool proposes outcomes (accuracy, and future AAA damage);
   the GM always finalises. This is the core design contract.
3. **The 9-line is roleplayed**, not a form. The tool holds supporting info only.
4. **No live game integration** for radio calls (see §2).
5. **Malfunction rates are per-aircraft, per-pass.** Verified correct (200k samples: 3.02% hung
   vs 3% configured; 4.60% dud vs 4.5%). A 4-ship flight showing ≥1 malfunction ~27% of the time
   is expected compounding, *not* a bug. Don't "fix" it; lower the sliders if it feels frequent.
6. **Commit and push after every change.**

---

## Reference sources

- [MCWP 3-23.1 Close Air Support](https://www.trngcmd.marines.mil/Portals/207/Docs/TBS/MCWP%203-23.1%20Close%20Air%20Support.pdf) — check-in brief format
- [SOFREP — what CAS is and isn't](https://sofrep.com/fightersweep/what-close-air-support-is-and-isnt-part-two/)
- [Mitchell Institute — US fixed-wing losses 1962–73](https://www.mitchellaerospacepower.org/us-fixed-wing-aircraft-losses-of-the-vietnam-war-1962-1973/)
- [HistoryNet — North Vietnam's light AAA](https://historynet.com/north-vietnams-light-anti-aircraft-artillery/)
- [Unsung vehicles](https://www.armanam.eu/vehicles.html) / [Prairie Fire vehicles](https://armedassault.fandom.com/wiki/S.O.G._Prairie_Fire_Vehicles) — airframe inventory
