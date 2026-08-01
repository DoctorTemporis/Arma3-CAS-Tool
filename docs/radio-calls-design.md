# Radio Prompt Cards — design plan

> **STATUS: DRAFT — awaiting GM approval. Do not implement until signed off.**
> Open decisions are listed at the bottom; several change the build.

## Scope (settled)

On-screen text the GM reads aloud, generated live from flight state. **No game integration** —
a sandboxed single HTML file cannot read Arma or inject audio into TFAR/ACRE2 (see ROADMAP §2).

**The problem being solved:** the GM voices the control agency *and* every pilot, while running
the mission and watching the game. Composing a correct check-in — current ordnance, current
playtime, right callsign — under that load is where the friction is. The tool already holds all
of it.

**Design rule:** cards must be *glanceable and short*. A paragraph won't get read aloud mid-session.
If a card can't be spoken in one breath, it's too long.

---

## Must-have features

### M1. Live data binding
Every call is generated from the flight's **current** state, never the state at generation.
If a flight has expended half its bombs, the check-in says so. This is the single biggest win —
it removes the "what did I say they had?" reconciliation entirely.

### M2. Speakable output (phonetic)
Numbers render as spoken words, matching the existing callsign style:
- `25 min` → "two five minutes"
- `4× Mk-82` → "four by Mark eight-two"
- `20mm` → "twenty mike-mike"

Requires a **`spoken` field on each weapon** in the library (e.g. `mk82` → "Mark eight-two",
`ffar` → "two point seven five rockets"). Defaults shipped; editable in the Weapon editor.

### M3. The call set
Proposed minimum set, mapped to workflow moments the tool already models:

| # | Call | Speaker | Fires when | Data used |
|---|------|---------|-----------|-----------|
| C1 | **Options offer** | Control → FAC | Options generated | All available: type, ships, ETA, playtime, load |
| C2 | **Check-in** | Flight → FAC | `AWAITING CHECK-IN` | Ships, type, ordnance, playtime, (abort code) |
| C3 | **In hot / attack run** | Flight → FAC | Employing | Weapon, attack heading |
| C4 | **Off / effects** | Flight → FAC | After release | Outcome already recorded |
| C5 | **Winchester** | Flight → FAC | All stores gone | — |
| C6 | **Bingo / RTB** | Flight → FAC | Playtime expiring | Remaining playtime |
| C7 | **Taking fire** | Flight → FAC | AAA toggled on | — |

C1 and C2 are the high-value pair. C3–C7 are one-liners and cheap once the engine exists.

### M4. Editable templates
Templates live in the Library, persisted, with a reset-to-defaults. Token-based:
`{CALLSIGN} {FLIGHT} {CONTROL} {SHIPS} {TYPE} {CONFIG} {ORDNANCE} {PLAYTIME} {ETA} {ABORT}`
Non-negotiable: the GM must be able to rewrite phrasing without touching code.

### M5. Contextual surfacing (not a menu to hunt through)
The tool knows the flight's state, so it should offer **the call you need now**:
- A `📻` button on each flight card opens that flight's radio panel, with the
  state-appropriate call **pre-expanded at the top**.
- The existing **alert bar** gains a `📻` shortcut when an alert implies a call
  (arrival → check-in, bingo → RTB, winchester → winchester).
- **REQUEST AIR** gets a single `📻 Read options to FAC` button for C1, covering all
  pending options in one script — this is the mouthful that most needs generating.

This is the difference between a feature that reduces load and one that adds a menu.

---

## UI design

### Flight radio panel (modal, consistent with existing editors)

```
┌─ RADIO — BANDIT FLIGHT ──────────────────────────── ✕ ─┐
│ ★ KNOWN • ELITE • SCAF • 2× A-1 Skyraider • Sandy      │
│                                                         │
│ ▼ CHECK-IN                              [copy]          │
│   ┌───────────────────────────────────────────────┐    │
│   │ "Monkey Mountain, BANDIT FLIGHT, flight of     │    │
│   │  two Skyraiders, four by Mark eight-two,       │    │
│   │  two by napalm, twenty mike-mike,              │    │
│   │  playtime two five."                           │    │
│   └───────────────────────────────────────────────┘    │
│                                                         │
│ ▶ IN HOT          ▶ WINCHESTER      ▶ TAKING FIRE      │
│ ▶ OFF / EFFECTS   ▶ BINGO / RTB                        │
└─────────────────────────────────────────────────────────┘
```

- Current-state call auto-expanded; others collapsed one-liners, click to expand.
- Monospace, large, high contrast — readable at a glance while talking.
- Regenerate button where a call has variable phrasing.

### Options offer (REQUEST AIR)
One script listing every pending option in sequence, so the GM can read the whole
availability call in one go rather than composing from three cards:

> "Sunray, Monkey Mountain. I have two flights for you. First, BANDIT FLIGHT, two Skyraiders,
> Sandy fit, overhead in one zero minutes, two five minutes playtime. Second, VOODOO SIX ONE,
> single Skyhawk, snake and nape, overhead in four minutes, one two minutes playtime. Say preference."

---

## Build phases

| Phase | Contents | Effort |
|-------|----------|--------|
| P1 | Template engine, token resolver, phonetic/spoken layer, weapon `spoken` field + defaults | M |
| P2 | C1 options offer + C2 check-in, flight radio panel, REQUEST AIR button | M |
| P3 | C3–C7 short calls, alert-bar shortcuts | S |
| P4 | Template editing UI in Library | S |

P1+P2 is the meaningful deliverable; P3/P4 are quick follow-ons.

---

## Risks

- **Cards too long** → won't be read. Mitigation: hard brevity rule, GM-editable.
- **Robotic sameness** → every flight sounds identical. Mitigation: optional flavour variants (D1).
- **Spoken-name maintenance** → every new weapon needs a `spoken` value. Mitigation: fall back to
  the display name when absent; never block.
- **Low actual value** → if the GM already rattles calls off fluently, this is wasted effort.
  This was flagged before and remains the biggest risk; P1+P2 first keeps the bet small.

---

## Open decisions (need GM input)

- **D1 — Accent/flavour variants.** ChelteNam locals do bad West Country. Should SCAF flights get
  West Country phrasing ("moi lover", "proper job", tractor references) and US flights straight
  US phraseology? Adds real character; roughly doubles the default templates.
- **D2 — Abort codes.** Real check-in briefs include one. Auto-generate a code word per flight
  and include it in C2? Nice authenticity, tiny cost, one more thing to say.
- **D3 — Call set trim.** Are C3–C7 wanted, or is C1+C2 the whole job? Cutting them halves P3.
- **D4 — Copy button.** Useful only if calls sometimes get pasted into Discord/TeamSpeak text;
  pointless if always spoken. Keep or drop?
- **D5 — 9-line readback.** The pilot reads back the FAC's 9-line. The tool deliberately doesn't
  hold 9-line data. Offer a generic readback skeleton, or leave entirely to roleplay?
- **D6 — SCAF comprehension.** Should lower-skilled SCAF pilots occasionally produce garbled or
  confused readbacks as flavour (and a mechanical hint about their skill)? Fun, but adds
  randomness to something the GM may want deterministic.
