# ChelteNam CAS / FAC Tracker

A single-file, offline tool for running **close air support** during Tactical Distractions'
Arma 3 sessions — the *ChelteNam* campaign, a West Country reimagining of Vietnam-era air war
(SCAF instead of VNAF, Chelt Cong instead of Viet Cong, Chairman Moo's Dorset, and so on).

Built for one GM to run live: a forward air controller (the player) calls for immediate CAS,
the tool generates realistic on-call options, tracks each flight from tasking through weapons
release to RTB, and now gives the GM spoken lines to read for every radio call involved.

No server, no database, no install. **Download `CAS-Tracker.html` and open it in any browser.**

---

## Quick start

1. Grab the latest [release](../../releases/latest) (or just `CAS-Tracker.html` from this repo).
2. Double-click it. It opens in your default browser and runs entirely client-side.
3. Everything auto-saves to that browser's local storage as you go. Use **SESSION → Export**
   to back up or move a session to another machine.

No accounts, no network access, nothing to configure before your first session.

---

## What it does

**On-call CAS, not pre-planned strikes.** The FAC requests immediate support; the tool rolls
a handful of realistic options — airframe, loadout configuration, pilot skill, fuel state,
ETA, playtime — reflecting aircraft diverted from other tasking, exactly as it would be in
the field. The FAC picks one (or several), and it's tracked through:

```
AVAILABLE → EN ROUTE → AWAITING CHECK-IN → ON STATION → (employ weapons) → RTB / departed
```

**Per-aircraft or per-flight ordnance**, your choice. Track exactly what each aircraft in a
section is carrying and let the GM pick which specific aircraft release on a given pass — or
pool ordnance at the flight level for a simpler game. A one-click **"employ all remaining"**
handles the common "expend everything you've got on this run" call.

**The tool suggests, the GM decides.** Every weapon release gets a suggested outcome — hit,
or a miss with cardinal direction and distance — weighted by pilot skill, weapon type,
weather, and whether the aircraft is under fire. The GM can accept it or override anything.

**A persistent roster of known units.** Recurring callsigns — BANDIT FLIGHT, GREASEMONKEY,
THUNDERBOLT — survive across sessions with sortie counts, a fixed personality, and (for
elite units) exemption from the standard SCAF accuracy penalty. Any flight generated during
a session can be promoted straight onto the roster with one click.

**Radio prompt cards.** The GM is voicing the control agency and every pilot while running
the mission — this generates the actual lines to read, live from current flight state, in
one of eight regional voices (or a fixed character voice for known units), including the
full attack-run exchange ("in from the ‹direction›" / "five seconds" / *cleared hot* /
"weapon away" / "off ‹direction›"). All of it — voices, phrasing, quips, abort-code words —
is editable in the **RADIO** tab.

**Alerts that survive a backgrounded tab.** Audible and visual warnings for bingo fuel,
arrival, and winchester, built on wall-clock timing rather than assuming the browser ticks
reliably while Arma has focus (it doesn't).

**A fully editable library.** Airframes, weapon loadout configurations, spawn weights, the
known-unit roster, and the entire radio vocabulary all live in one editable, exportable
library — tune it to your group without touching the file's code.

---

## Tabs

| Tab | Purpose |
|---|---|
| **OPS BOARD** | Active and completed flights, live timers, weapon employment |
| **REQUEST AIR** | Generate on-call options; assign one to the FAC |
| **TUNING** | Every generation/accuracy parameter — sliders, not code |
| **AFTER-ACTION LOG** | Timestamped record of the session; exports to `.txt` |
| **LIBRARY** | Airframes, weapon loadouts, spawn weights, known-unit roster |
| **RADIO** | Voices, call templates, ChelteNam moments, spoken vocabulary, abort words |
| **SESSION** | Export/import, new session, full reset |

---

## Design principles

- **Single file, no dependencies.** Everything — logic, styling, even the alert tones — is
  generated client-side. It has to survive being handed to a co-GM with nothing but a browser.
- **Suggest, never auto-apply.** The tool proposes outcomes; the GM always has final say.
- **Reduce load, don't add it.** Anything that would require GM data-entry *during* a strike
  (e.g. setting attack headings) is deliberately left as a spoken placeholder instead of app
  state — the point is fewer things to manage mid-session, not more.
- **On-call, not scripted.** Fuel state, loadout, and pilot skill are rolled per appearance,
  even for known recurring units — nothing is fixed except identity.

See [`ROADMAP.md`](ROADMAP.md) for planned features, standing constraints, and the reasoning
behind past design decisions. Feature-specific design docs live in [`docs/`](docs).

---

## Setting

**ChelteNam** is Tactical Distractions' in-house reskin of the Vietnam War, built to keep the
tone comedic and sidestep real-world historical/racial baggage: the Chelt Cong, Ho Cheese
Minh, Cambodia-as-Derbyshire, Chairman Moo's Dorset, and pervasive bad West Country accents.
South Vietnamese Air Force becomes **SCAF** — the South ChelteNam Air Force — throughout,
including in the underlying data model.

---

## Requirements

Any reasonably modern desktop browser (Chrome, Edge, Firefox). No internet connection needed
after the first download. Audio alerts require one click/keypress to unlock, per standard
browser autoplay policy — a **Test sound** button in TUNING confirms it's working.
