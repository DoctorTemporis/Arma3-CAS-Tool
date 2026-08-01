# Radio Prompt Cards — design plan (rev 2)

> **STATUS: APPROVED.** D1–D6 and Q1–Q4 all answered; building P1–P4.
> Q1: radio panel only (Employ modal stays uncluttered). Q2: voice list accepted, extend later.
> Q3: character voices — BANDIT stereotypical but fearless; GREASEMONKEY surly;
> THUNDERBOLT is Lt Col Studebaker-Mackenzie, extreme evangelical, smiting heathens.
> Q4: ChelteNam moments extend to US voices but only very occasionally.
> **P5 (template editing UI) deferred** — templates ship as editable data, UI to follow.

## Scope (settled)

On-screen text the GM reads aloud, generated live from flight state. **No game integration** —
a sandboxed single HTML file cannot read Arma or inject audio into TFAR/ACRE2 (ROADMAP §2).
All calls are spoken over ACRE2 in game, so **no copy button** (D4: declined).

**Problem being solved:** the GM voices the control agency *and* every pilot while running the
mission. Composing a correct check-in — current ordnance, current playtime, right callsign — under
that load is the friction. The tool already holds all of it.

**Design rule:** cards are glanceable and short. If a card can't be spoken in one breath, it's
too long.

---

## Decisions taken (D1–D6)

| # | Decision | Outcome |
|---|----------|---------|
| D1 | Accents | **Yes — and regionally varied within US**, not just US vs SCAF |
| D2 | Abort codes | **Yes** — generated per flight, spoken in check-in |
| D3 | Call set | **All of it** |
| D4 | Copy button | **No** — always spoken over ACRE2 |
| D5 | 9-line | **Roleplay only.** No 9-line data. Talk-on + ordnance + direction in/out is the real flow |
| D6 | SCAF comedy | **Yes** — confusion quips, GM-supplied examples below |

---

## Voices (D1)

Each flight is assigned a **voice** at generation, shown on the card as a tag (e.g. `🗣 ALABAMA`)
so the GM knows which accent to attempt. Voices produce genuinely different phrasing — not one
shared template with a catchphrase bolted on.

**US voices** (random per flight): `ALABAMA`, `TEXAS`, `KANSAS`, `CALIFORNIA`, `NEW JERSEY`
**SCAF voices** (West Country, random per flight): `SOMERSET`, `DEVON`, `BRISTOL`

**Known units keep a fixed voice.** BANDIT must sound like BANDIT every time — that consistency
is the relationship the group has with them. Roster entries gain an optional `voice` field;
blank = random per appearance.

Illustrative check-in, same flight, three voices:

- **KANSAS** — "Monkey Mountain, VOODOO SIX ONE. Two Skyhawks. Four Mark eight-twos, two napalm,
  twenty mike-mike. Playtime one two. Abort code HAYSTACK."
- **CALIFORNIA** — "Monkey Mountain, VOODOO SIX ONE cruisin' in, couple of Skyhawks. We got four
  Mark eight-twos, pair of napalm and the twenty mike-mike. One two minutes of playtime.
  Abort's HAYSTACK."
- **SOMERSET** — "Mornin' Monkey Mountain, this'm VOODOO SIX ONE, two of us. Got four gurt Mark
  eight-twos, couple o' napalm an' the twenty mike-mike. Playtime's one two minutes, moi lover.
  Abort word's HAYSTACK."

---

## The call set (D3 — all)

| # | Call | Speaker | Surfaces when |
|---|------|---------|---------------|
| **R1** | **Options offer** | Control → FAC | Options generated (REQUEST AIR) |
| **R2** | **Check-in** | Flight → FAC | `AWAITING CHECK-IN` |
| **R3** | **Talk-on / tally** | Flight → FAC | On station, pre-attack |
| **R4** | **Attack run** *(sequenced)* | Flight → FAC | Employing |
| **R5** | **BDA query** | Flight → FAC | After a release |
| **R6** | **Winchester** | Flight → FAC | All stores expended |
| **R7** | **Bingo / RTB** | Flight → FAC | Playtime expiring |
| **R8** | **Taking fire** | Flight → FAC | AAA toggled on |
| **R9** | **ChelteNam moment** | SCAF flight | On demand / random (D6) |

### R4 — Attack run, as the group actually runs it (D5)

Not a one-liner. A **stepped sequence** matching the real exchange, with the FAC's line shown
greyed for context since that's the *player's* to say:

```
1. IN        "VOODOO SIX ONE, in from the south."
2. 5 SEC     "Five seconds."
3. ─ FAC ─   "Cleared hot."                     ← player's line, greyed
4. AWAY      "Weapon away."   (or bombs/rockets/guns away, per store)
5. OFF       "Off to the west."
```

- **Direction in / out are GM-settable** (cardinal dropdowns) since the FAC passes them.
  Default: random ingress with a roughly opposite egress; remembered per flight between runs.
- **Abort code** available as a step-3 alternative for waving the run off.
- Step 4 wording follows the actual weapon employed (bombs / rockets / guns away).

### R9 — ChelteNam moments (D6)

Comedy interjections for SCAF flights, weighted toward lower-skilled pilots. GM-supplied seeds:

- *Ordnance* — "Well, I got some big bombs and some whooshy stuff, mebbe rockets."
- *Navigation* — "Can you tell me where that is relative to my uncle Jeb's goat farm?"
- *Direction* — "Which way is east again? Oh — way the sun comes up."
- …plus a starter pool of similar oddities, **editable and extensible in the Library**.

**Surfacing:** never silently replaces a call. The card shows the straight version with a
`🎲 ChelteNam moment` button to swap one in, **plus** an optional auto-chance (tuning slider,
default low) so it can surprise the GM too. Reroll and revert-to-straight always available.
Categories are context-aware — an ordnance quip appears on check-in, a direction quip on the
attack run.

---

## Speakable output

Numbers render as spoken words, matching existing callsign style: `25 min` → "two five minutes";
`4× Mk-82` → "four by Mark eight-two"; `20mm` → "twenty mike-mike".

Requires a **`spoken` field on each weapon** in the library (defaults shipped, editable in the
Weapon editor). Falls back to the display name when absent — never blocks.

**Abort codes (D2):** one word drawn per flight from a word pool (`HAYSTACK`, `CIDER`,
`MILKCHURN`…), fixed for that flight's life, spoken in check-in and reusable to wave off a run.

---

## UI design

### Flight radio panel — `📻` button on each flight card

```
┌─ RADIO — BANDIT FLIGHT ─────────────────────── ✕ ─┐
│ ★ KNOWN • ELITE • SCAF • 🗣 SOMERSET               │
│ 2× A-1 Skyraider • Sandy • abort word CIDER        │
│                                                     │
│ ▼ CHECK-IN                        [🎲 ChelteNam]   │
│  ┌──────────────────────────────────────────────┐ │
│  │ "Mornin' Monkey Mountain, this'm BANDIT       │ │
│  │  FLIGHT, two of us. Four gurt Mark eight-     │ │
│  │  twos, couple o' napalm, twenty mike-mike.    │ │
│  │  Playtime two five. Abort word's CIDER."      │ │
│  └──────────────────────────────────────────────┘ │
│                                                     │
│ ▶ TALK-ON   ▶ ATTACK RUN   ▶ BDA   ▶ WINCHESTER   │
│ ▶ BINGO/RTB ▶ TAKING FIRE                          │
└─────────────────────────────────────────────────────┘
```

- **Current-state call auto-expands** — arrival shows check-in, bingo shows RTB, etc.
  The tool offers the call you need now rather than a menu to hunt through.
- Others collapsed to one-line headers; click to expand.
- Large monospace, high contrast — readable while talking.

### Attack run in the Employ modal
The run patter belongs where the GM already is when a weapon goes down, so a **compact run strip**
appears in the Employ modal (in-heading / out-heading selectors + the five steps), in addition to
the full sequence in the radio panel. *(See Q1 — confirm you want it in both places.)*

### Options offer — REQUEST AIR
One button scripting **all** pending options in sequence, in the control agency's voice:

> "Sunray, Monkey Mountain. Two flights for you. First, BANDIT FLIGHT, two Skyraiders, Sandy fit,
> overhead in one zero minutes, two five minutes playtime. Second, VOODOO SIX ONE, single Skyhawk,
> snake and nape, overhead in four minutes, one two minutes playtime. Say preference."

### Alert-bar shortcuts
Existing alert bar gains a `📻` shortcut jumping straight to the implied call
(arrival → check-in, bingo → RTB, winchester → winchester).

---

## Data model additions

- **Flight:** `voice`, `abortCode`, `runIn` / `runOut` (attack headings, remembered between runs)
- **Weapon:** `spoken`
- **Roster unit:** optional fixed `voice`
- **Library:** `voices` (templates per call type per voice), `quips` (ChelteNam moments), `abortWords`
- **Tuning:** ChelteNam-moment auto-chance
- **Migration:** all additive with fallbacks; existing sessions keep working

---

## Build phases

| Phase | Contents | Effort |
|-------|----------|--------|
| **P1** | Template engine + token resolver, phonetic/spoken layer, weapon `spoken` defaults, voice assignment, abort codes | M |
| **P2** | R1 options offer + R2 check-in, flight radio panel, REQUEST AIR button, alert-bar shortcuts | M |
| **P3** | R4 attack run sequence (panel + Employ strip), R3/R5–R8 short calls | M |
| **P4** | R9 ChelteNam moments + quip library | S |
| **P5** | Template/voice editing UI in Library | S |

P1+P2 is the first meaningful deliverable and proves the concept before the content bulk lands.

---

## Risks

- **Content volume.** 9 call types × 8 voices ≈ 70+ short strings. Mitigation: ship a strong
  default set, make everything editable, fall back to a neutral voice if a template is missing.
- **Cards too long** → won't be read aloud. Mitigation: hard brevity rule; accents add colour
  but must not add length.
- **Accent parody landing badly** — mitigated by it being the group's own established humour,
  and fully editable.
- **Low actual value** if the GM already rattles calls off fluently. Smallest-bet mitigation:
  ship P1+P2, use it for one session, then decide on P3–P5.

---

## Remaining questions

- **Q1 — Attack-run placement.** Full sequence in the radio panel *and* a compact strip in the
  Employ modal, or panel only? The Employ modal is already busy with per-aircraft rows.
- **Q2 — Voice list.** Are `ALABAMA / TEXAS / KANSAS / CALIFORNIA / NEW JERSEY` the right US set,
  and do you want SCAF split into `SOMERSET / DEVON / BRISTOL` or kept as one West Country voice?
- **Q3 — Fixed voices for the three known units.** BANDIT, GREASEMONKEY, THUNDERBOLT — any
  specific voices you already hear in your head for them?
- **Q4 — ChelteNam moments for US voices too?** Currently SCAF-only. A laid-back California pilot
  being unhelpfully vague is also funny — extend, or keep confusion strictly SCAF?
