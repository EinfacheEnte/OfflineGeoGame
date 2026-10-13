# OfflineGeoGame

A lightweight offline Android party game: draw a clue for a country, guess the country, and place a pin on a world map. Closer guesses earn more points.

**Status:** early planning draft, updated 2026-10-13. The game is not implemented. Recommendations below are proposals awaiting discussion, not agreed requirements.

## Goals from the initial brief

- Make a fun game for friends playing together in person.
- Keep the app small and straightforward to build.
- Play without internet access.
- Use one phone per player; friends join the same nearby party.
- Give one player a country name and let them draw a clue for the others.
- Let the other players identify the country and pinpoint it on a map.
- Award points based on the distance between the guess and the correct country.
- Explore a nearby party connection, preferably Bluetooth if practical.
- Assess support for more than two players realistically.

## Current project

The repository contains a Kotlin / Jetpack Compose Android starter with Material 3 and template tests. Its configured minimum Android version is Android 7.0 (API 24). It has no game logic, drawing tools, geography data, map, or multiplayer implementation yet.

The SDK and dependency versions are existing project settings, not a validated build environment. This planning work does not establish that the current starter builds successfully.

## Suggested game flow

This is a discussion starting point; timing, scoring, and guessing rules remain open.

1. A player creates a party. Friends join and choose nicknames.
2. The host starts a game using the agreed round settings.
3. The current drawer privately sees a country name.
4. The drawer sketches a clue; the other players see the drawing.
5. Each guesser places a map pin and submits a guess. Whether a country-name answer is also required is undecided.
6. The round ends when the timer expires or everyone has submitted, depending on the chosen rules.
7. Everyone sees the target country, submitted pins, distances, and points.
8. The drawing role rotates. After the agreed number of turns, show the final standings.

The host and drawer are separate roles: the host can participate and remains host while other players draw.

## Agreed decisions

- **One phone per player.** Each person uses their own Android phone in the connected party. Shared-phone and both-mode options were considered; the user chose individual phones. Pass-and-play is not required for the first version.

## Realistic multiplayer assessment

**More than two players is feasible.** A host-and-clients arrangement suits this game: one phone coordinates the party, receives drawings and guesses, and distributes updates.

Recommended development target: **2–4 players total, including the host**, with two-device communication proved first and four-player support tested before release. This is a proposed validation target, not a measured capacity or an Android guarantee. Consider 6–8 players only after tests justify that scope.

| Option | Advantages | Costs / limitations | Initial judgment |
| --- | --- | --- | --- |
| Native Bluetooth Classic | No internet, router, or Google Play services dependency; app controls communication | Pairing, discovery, multiple connections, permission handling, and recovery need custom work; actual capacity varies by device | Worth exploring if Bluetooth-only or avoiding Google services is a firm requirement |
| Nearby Connections with a star topology | Supports one host with multiple clients; handles nearby discovery and transport; works without internet | Uses Bluetooth, BLE, and Wi-Fi rather than guaranteeing Bluetooth-only; Android SDK depends on Google Play services; adds dependencies and permission handling | Leading option if ease of development and group play matter most |
| Local Wi-Fi / hotspot | No internet needed; enough bandwidth for drawing updates | Requires local network setup and discovery; hotspot behavior varies by phone | Possible alternative if network setup is acceptable |
| One shared phone | No pairing or connection failures; smallest implementation | Players pass the phone; simultaneous private guesses are awkward; drawing must be hidden from the drawer's country prompt | Alternative mode if it fits the intended experience |

Android supports managing multiple Bluetooth connections, but its ordinary RFCOMM sample is a two-device example. The documented channel model permits one connected client per RFCOMM channel; multi-client support needs appropriate separate channel/service handling. It is not enough to assume that a single copied sample accepts an entire party. Do not advertise a fixed Bluetooth player limit from a theoretical radio limit.

For Nearby Connections, use a host-and-clients star rather than a full mesh. Internet connectivity is unnecessary, but the relevant radios and permissions still need to be available. Whether requiring Google Play services is acceptable is an open decision.

### Bluetooth-only issues and combined-transport benefits

The combined option discussed here means **Google Nearby Connections**, not a second custom networking implementation. It manages nearby communication using Bluetooth, BLE, and Wi-Fi. Both options can work without internet access or a router.

| Concern | Bluetooth-only | Nearby Connections using Bluetooth and Wi-Fi |
| --- | --- | --- |
| Group connections | We manage separate client connections and channel/service handling; practical capacity needs testing across phones | Provides a star topology with one host and multiple clients; actual supported group size still needs testing |
| Live drawing | Compact stroke messages should be manageable, but the host relays updates to each guesser and traffic grows with the group | Wi-Fi provides more bandwidth and headroom for live updates and drawing synchronization |
| Joining | We implement discovery, pairing, connection requests, timeouts, and errors | Handles much of discovery and connection setup; the app still needs a lobby, permissions, and connection confirmation |
| Development effort | More custom networking code and device-specific debugging | Less low-level connection work, leaving more development time for game behavior |
| App size | Avoids an additional networking SDK | Adds a dependency; measure its effect on release download size rather than estimating it |
| Compatibility | Can support phones without Google Play services and does not need Wi-Fi enabled | The proposed Android SDK requires Google Play services and availability of the necessary radios |
| Disconnects | We implement reconnection and game-state recovery | Provides connection events and transfer handling; reconnect policy and restoring the round remain app responsibilities |

Specific Bluetooth-only work to account for:

- **Multiple clients:** the ordinary two-phone sample is not a complete party implementation. Track each connection, handle channel/service allocation, and communicate with every player separately.
- **Discovery during play:** Android documents that Bluetooth discovery slows connections. Joining during a round must account for discovery potentially affecting existing communication.
- **Drawing synchronization:** send ordered drawing actions, including undo and clear, and restore a complete drawing after reconnecting. These game-level requirements also apply to Nearby.
- **Real-phone validation:** working on two devices does not establish reliability for four or six. Test different manufacturers and Android versions before advertising a limit.

Bluetooth-only remains a reasonable option for a small group using compact sketches. Its benefits include fewer dependencies, no Wi-Fi requirement, and support for phones without Google Play services. Drawing bandwidth alone is not a reason to reject it.

The combined option's main benefit is reducing custom connection handling while adding bandwidth headroom. It does not solve game rules, timers, private guesses, scoring, or host departure. It also does not guarantee a player limit or better battery life. Google documents SDK usage diagnostics controlled by the device's Usage & diagnostics setting.

**Recommendation, not an agreed decision:** use Nearby Connections if ease of development and group play are the priorities, initially validating four players total. Prefer exploring Bluetooth-only if avoiding Google Play services or keeping Wi-Fi disabled is a firm requirement. The transport choice remains open.

Before choosing a transport, run a small connection experiment on physical phones. Check joining, all phones taking the drawing role, live drawing latency, and connection loss. Emulator testing alone cannot establish reliability.

## Keeping the app lightweight

Recommended direction:

- Keep the existing Kotlin / Compose foundation.
- Bundle a simplified vector world map and country data so the first game works offline immediately after installation.
- Draw the map directly with a canvas, with pan and zoom. Avoid downloading map tiles or adding a large map SDK unless requirements call for one.
- Keep country identifiers, display names, and scoring geometry in the bundled data. The visual map and scoring geometry must agree.
- Start with a small drawing toolset: pen, a few colors, undo, and clear. Confirm the final tools during planning.
- Transmit compact, normalized stroke coordinates and drawing actions in batches rather than screenshots for every update.
- Keep party state in memory initially. Saved history and reconnect persistence are separate scope decisions.
- Measure a release build's download and installed size after the first playable version; do not promise an APK size before measurement.

Country boundaries, small islands, map simplification, and dataset attribution need deliberate handling. A simplified map must not make an included country impossible to find or score correctly. Choose the dataset, included territories, and disputed-border treatment before implementation.

## Geography and scoring decisions

"Distance to the country" has several possible meanings:

| Method | Player experience | Implementation implications |
| --- | --- | --- |
| Country area | Any pin inside the country earns full location points; outside guesses lose points with distance to the nearest country boundary | Needs point-in-country checks and geographic distance to polygon boundaries, including islands and date-line cases |
| Capital | Players try to locate the capital | Easier: one reference coordinate per country; changes the challenge from locating the country to locating its capital |
| Representative point | Players aim at a displayed or explained reference point | Easier distance calculation, but a geographically correct pin can lose points; the target must be understandable |

**Recommendation for discussion:** use the country's area if the intended challenge is locating a country. Use a capital only if capital knowledge is part of the game. A naive centroid can lie outside a country or unfairly penalize valid guesses.

Calculate distances geographically, in kilometers, rather than using pixel distance on the map. Handle the international date line and multiple land areas. For country-area scoring, give an inside pin zero error distance; outside distance is to the nearest included land boundary.

Example scoring curve, subject to approval: `points = round(1000 * exp(-distanceKm / 1500))`. This gives 1000 points at zero distance and progressively fewer points farther away. Its parameters, any cutoff, name-guess points, time bonuses, drawer points, and difficulty balancing are all undecided. The curve is illustrative, not a final game rule.

The game must also decide whether naming the country is a separate scored answer or something players infer while placing their pin. Scoring both creates a different experience from scoring location alone.

## Proposed implementation stages

These are planning milestones, not a commitment to implement before the open decisions are resolved.

1. **Agree on the playable rules.** Decide device mode, connection constraints, intended group size, round flow, country set, guessing, and scoring. Write examples of a full round.
2. **Prove the connection.** On physical phones, create and join a party and exchange small messages. Start with two phones, then test the proposed group size. Compare candidates if the preferred transport fails the requirements. Finalize the transport from evidence.
3. **Build a local game core.** Implement country selection, drawing, the offline map, pin placement, distance calculation, round transitions, and results on one device. This is a development step; it does not imply a shipped pass-and-play mode.
4. **Connect the game.** Let the host control rounds and scores. Send the country prompt only to the drawer, drawing updates to guessers, and private guesses to the host. Reveal all guesses at the agreed time.
5. **Handle normal party interruptions.** Add the agreed joining, disconnect, timer, and host-exit behavior. A simple first version may end the party when its host leaves; automatic host migration is additional work.
6. **Validate and trim.** Play complete games on representative phones without internet, check fairness and map usability, measure release size and drawing traffic, then decide whether a larger group is worthwhile.

Suggested internal responsibilities: game rules and state; geography and scoring; drawing and map UI; nearby communication. Keep game rules independent of the transport so they can be checked without radio connections.

For friendly in-person play, a host-controlled game is a reasonable trust model. The host will know the target and calculate results; sophisticated protection against a modified host app would substantially expand scope.

## Proposed first-version acceptance checks

Finalize these after the product decisions:

- A fresh installation can start a game without downloading maps or country data.
- Players create and join a party and complete a match with no internet connection, using the chosen local connection.
- The agreed number of players can each take a drawing turn, including a drawer who is not the host.
- Only the drawer sees the country prompt during the round in the ordinary app UI.
- Drawing updates remain usable on the tested devices; measure latency before setting a numeric target.
- Guesses remain private until the reveal, if that rule is selected.
- Scoring behaves as the chosen rule says for inside-country pins, nearby misses, islands, and date-line cases.
- Everyone receives the same scores and round results from the host.
- A lost connection or host exit follows the documented rule without leaving the party stuck.
- Map zoom and pin placement work for the smallest included countries on a phone screen.
- A measured release build meets the size budget once that budget is decided.

## Decisions to work through together

Answer by number if useful. We can work through these in rounds; no need to decide everything at once. Recommendations are starting points, not selected answers.

### Playing together

1. **Device mode — resolved:** everyone uses their own phone. The first version focuses on a connected party.
2. **Offline connection:** must it be Bluetooth-only, or is automatic Bluetooth + local Wi-Fi acceptable if no internet or router is needed?
3. **Google services:** is requiring Google Play services acceptable, or must the app also work on phones without them?
4. **Group size:** how many friends do you normally expect—2, 3–4, 5–6, or more? What is the smallest launch version you would actually enjoy using?
5. **Setting:** will this be played at a table, outdoors, while traveling, or somewhere else? This affects range, screen visibility, and interruptions.

### Drawing and guessing

6. **Drawing timing:** should others watch strokes appear live, or see only the finished drawing? Live viewing feels more interactive; finished drawings reduce synchronization work.
7. **Clue rules:** can players draw flags, country outlines, landmarks, letters, numbers, or the country's written name? Which are allowed?
8. **Country assignment:** one random country, a choice of three, or host-selected countries? Can the drawer skip a country they do not know?
9. **Answers:** place a pin only, or also submit a country name? If names are required, select from a list or type freely?
10. **Submission:** one locked guess, editable until the timer ends, or multiple attempts? Should correct guessers stop seeing updates?
11. **Privacy:** keep other players' pins and answers hidden until the reveal, or show them during the round?
12. **Round timing:** one shared drawing/guessing timer or a drawing phase followed by a guessing phase? What duration would feel right?
13. **Early finish:** end when everyone submits, keep playing until the timer expires, or let the drawer finish manually?

### Points and fairness

14. **Location target:** any point inside the country, the capital, or a reference point? Country-area scoring best matches locating a country.
15. **Point sources:** location only, or separate points for the country name and response speed?
16. **Drawer points:** should the drawer earn points when others guess well, receive a fixed amount, or earn none? What incentive would make drawing fun rather than deliberately misleading?
17. **Winner:** individual scores or teams? How should ties be handled?
18. **Match length:** a fixed number of rounds, one or more drawing turns per player, or an open-ended game stopped by the host?

### Map and country set

19. **Country pool:** the whole world from launch, familiar countries first, or continent/difficulty options?
20. **Small places and territories:** include microstates and dependencies, or use a narrower clearly documented set? Is there a preferred country-list standard?
21. **Map hints:** borders only, continent names, country names, or hints after some time? Country labels can remove the name-guess challenge while still leaving the location challenge.

### Practical scope

22. **Phone support:** which Android phones can you and your friends test on? Is the existing Android 7 minimum useful, or is supporting newer phones enough?
23. **Size budget:** does lightweight mean a strict download limit, smooth performance on older phones, quick setup, or all three? Is there a concrete size target?
24. **Interruption rules:** when a player disconnects, pause, continue without them, or end the round? When the host leaves, is ending the party acceptable for version one?
25. **Distribution and language:** a private APK for friends or a Play Store release? English, German, or both?
26. **Local persistence:** should the app remember nicknames/settings, save match history or drawings, or forget everything when the party ends?
27. **Visual direction:** playful sketchbook, clean minimal interface, or another style? We can settle this after the game rules and connectivity.

## Candidate later features

Possible later additions, pending priorities: larger groups, teams, pass-and-play, continent packs, saved drawings, match history, and host migration. They are not first-version requirements yet. Online matchmaking, accounts, chat, and cloud storage are outside the current brief.

## Technical sources

Official documentation checked on 2026-10-13:

- [Android Bluetooth overview](https://developer.android.com/develop/connectivity/bluetooth): includes managing multiple connections.
- [Connecting Bluetooth devices](https://developer.android.com/develop/connectivity/bluetooth/connect-bluetooth-devices): RFCOMM client/server setup, pairing, channel constraints, and background work.
- [Nearby Connections overview](https://developers.google.com/nearby/connections/overview): offline nearby exchange over Bluetooth, BLE, and Wi-Fi; Google Play services SDK also documents usage analytics, which must be considered when defining privacy expectations.
- [Nearby Connections strategies](https://developers.google.com/nearby/connections/strategies): star topology supports one hub with multiple clients.

These sources support feasibility and the comparison above. They do not establish this app's player capacity, package size, connection reliability, or implementation schedule.
