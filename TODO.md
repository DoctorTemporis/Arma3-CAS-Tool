# Development notes — toward v3

## In progress: multiple loadout configurations per airframe

**Goal:** each airframe carries several *named* weapon configurations instead of one fixed load. Generation picks one (weighted); the flight records which config it arrived in; the library lets you edit multiple configs per airframe.

### Data model change
- Airframe `load: [...]` → `loadouts: [ { name, weight?, load: [...] }, ... ]`.
- Flight gains `configName` (the chosen config's name).
- `genFlight`: after choosing the airframe, `weightedPick(af.loadouts)`; build per-plane stores from `cfg.load`; store `configName`.
- `migrateState`: airframes with old `load` and matching a DEFAULT id are upgraded to the new default `loadouts`; unknown/custom airframes become a single `{name:'Standard', load}`. Flights without `configName` default to ''.
- Display `configName` on flight / available / closed cards and in the library list.
- Airframe editor: manage a list of configs (name + weight + station rows), add/remove config, add/remove station.
- Employ / per-plane accounting: unchanged (operates on `f.loadout`).

### Researched first-pass configs (Vietnam era, S.O.G. Prairie Fire + Unsung)
- **A-1 Skyraider:** Strike (Mk-82×6, M117×2, FFAR, 20mm); Sandy/RESCAP (Mk-82×4, CBU×4, napalm×2, FFAR, 20mm — danger-close SAR); Nape & Rockets.
- **A-4 Skyhawk:** Snake & Nape; Iron & Zuni; CBU.
- **A-6 Intruder:** Heavy Iron (Mk-82×18); Snake (Snakeye + Mk-83).
- **A-7 Corsair II:** Iron; CBU.
- **F-100:** Snake & Nape (2×Snakeye + 2×napalm + 20mm); Iron; Rockets.
- **F-105:** Heavy Iron (M117×6); Mk-82; CBU.
- **F-4:** Iron; CBU & Zuni; Nape.
- **A-37:** Shake & Bake (2×Mk-82 + 2×napalm); Rockets; CBU.
- **F-5:** Iron; Snake & Nape.
- **OV-10:** FAC/Marker (WP FFAR); Light Strike.
- **UH-1C gunship:** M21/Flex (miniguns + 2×7 rockets); Hog/XM3 (48 rockets); Heavy Hog (40mm + 2×19 rockets); Frog (40mm nose).
- **AH-1G Cobra:** Hog/Rockets (4×19 + turret); Guns (rockets + minigun pods); 20mm/XM35 (heavy cannon, fewer rockets).
- **ACH-47:** Guns-A-Go-Go (rockets + 20mm + minigun + 40mm).
- **OH-6A:** Scout (minigun + rockets).

Sources: Aviation Geek Club (A-1 Sandy load), Vietnam War Fandom / centaursinvietnam.org (Huey subsystems, Hog/Heavy Hog/Frog), globalsecurity.org + DTIC (AH-1G XM35), NASM / historynet / defensemedianetwork (F-100 snake&nape, A-4, A-37 shake&bake).

### After this feature
- Consider per-airframe guns becoming per-aircraft (currently flight-level coarse).
- Tag v3 once the loadout feature is validated in a session.
