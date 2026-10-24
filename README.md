# BORO_REALM // SYS_SPEC_26.3

```text
+--------------------------------------------------------------------------------------------------+
| SYSTEM_ID    : BORO_REALM_CROSSPLAY_CLUSTER                                                      |
| RUNTIME      : PaperMC 26.3 (GeyserMC + Floodgate + ViaVersion + ViaBackwards + CoreProtect)     |
| IPC_DAEMON   : boro-bridge (Python 3.12 RCON + Log Tail FSM + Async Operator Queue)              |
| AUDIT_STATUS : 45/45 STRUCTURAL COHESION INVARIANTS VERIFIED                                     |
| PORTAL_URI   : https://realm.boroghor.com/                                                       |
+--------------------------------------------------------------------------------------------------+
```

---

## 01 // NETWORK_INGRESS_MATRIX

Pure vanilla client protocol compatibility across Java Edition and Bedrock Edition (Windows, iOS, Android, Consoles). No client-side modifications or resource packs required.

| PROTOCOL_LAYER | INGRESS_HOST | PORT_BINDING | TRANSPORT_ENGINE | ROUTING_NOTES |
| :--- | :--- | :--- | :--- | :--- |
| `JAVA_PRIMARY` | `play.boroghor.com` | `25565/tcp` | Cloudflare DNS `_minecraft._tcp` SRV | Resolves automatically to active TCP tunnel endpoint. |
| `BEDROCK_PRIMARY` | `play.boroghor.com` | `18629/udp` | GeyserMC + Floodgate UDP | Explicit UDP port `18629` required on Bedrock clients. |
| `JAVA_DIRECT` | `bore.pub` | `43991/tcp` | Bore Rust TCP Forwarder (`boro-tunnel`) | Direct fallback when client resolver strips SRV records. |
| `BEDROCK_DIRECT` | `signaling-ala.tun.ply.gg` | `18629/udp` | Playit Tunnel Node | Direct UDP routing endpoint. |

---

## 02 // CONTAINER_TOPOLOGY_AND_IPC

Defined in `docker-compose.yml`. Four isolated services operate with shared volume mounts and localhost RCON IPC:

```text
[ PUBLIC_INTERNET ]
   |
   +---> play.boroghor.com (SRV _minecraft._tcp) ---> bore.pub:43991 ---> [ boro-tunnel ] (alpine:3.20)
   |                                                                             |
   +---> play.boroghor.com:18629/udp ---> signaling-ala.tun.ply.gg:18629 --------+
   |                                                                             |
   |                                                                             v
   |                                                                   [ boro-minecraft ]
   |                                                                   itzg/minecraft-server (Paper 26.3)
   |                                                                   - TCP 25565 (Java)
   |                                                                   - UDP 19132 (Geyser Bedrock)
   |                                                                   - TCP 25575 (RCON Local)
   |                                                                             ^
   |                                                                             | RCON (25575) + /data/logs
   |                                                                             v
   |                                                                    [ boro-bridge ]
   |                                                                    python:3.12-alpine (bridge.py)
   |                                                                    - Log Tailer (/data/logs/latest.log)
   |                                                                    - Gauntlet 30Hz Poll Loop (2.0s tick)
   |                                                                    - ACL Registry (/data/logs/survivors.json)
   |                                                                    - Operator Queue (/data/logs/operator_inbox.json)
   |
   +---> realm.boroghor.com (HTTPS/443) ---> Cloudflare Tunnel ---> [ boro-web ] (nginx:alpine :8767)
```

| CONTAINER_NAME | IMAGE | NETWORK_MODE | VOLUME_MOUNTS | PROCESS_ROLE |
| :--- | :--- | :--- | :--- | :--- |
| `boro-minecraft` | `itzg/minecraft-server:latest` | `bridge` (`25565/tcp`, `19132/udp`, `25575/tcp`) | `./data:/data` | Paper `26.3` JVM (`3G` heap), GeyserMC, Floodgate, ViaVersion, CoreProtect. |
| `boro-tunnel` | `alpine:3.20` | `service:mc-crossplay` | `none` | Executes `bore local 25565 --to bore.pub --port 43991` inside retry loop. |
| `boro-bridge` | `python:3.12-alpine` | `bridge` | `./bridge:/app`, `./data/logs:/data/logs` | Runs `python -u /app/bridge.py` for chat commands, Gauntlet trial checks, and operator inbox queue. |
| `boro-web` | `nginx:alpine` | `bridge` (`8767:80/tcp`) | `../boro-realm:/usr/share/nginx/html:ro` | Serves static technical specification portal to Cloudflare Tunnel. |

---

## 03 // BRIDGE_STATE_MACHINE_AND_IPC_COMMANDS

`bridge.py` tails `/data/logs/latest.log` and communicates with `boro-minecraft:25575` via RCON.

### 3.1 Player Classification & First-Join Lifecycle
- **Sovereign Operators (`OPS = {"steveXBT", ".rohG_oroB", "rohG_oroB"}`):**
  - Granted `gamemode survival` on join.
  - Non-command chat lines are serialized into `/data/logs/operator_inbox.json` (`timestamp`, `player`, `message`, `processed: false`) for asynchronous execution and in-game RCON `tellraw` response.
- **Unverified Guests (`player not in survivors.json`):**
  - First join initializes `gamemode adventure`, teleports to Alpine Sovereign Estate Grand Foyer (`-282 81 -348 -90 0`), sets `spawnpoint @s -282 81 -348`, and grants `minecraft:cooked_beef 16`.
  - Protected against block placement/destruction while touring the Estate.
- **Verified Survivors (`player in survivors.json`):**
  - Unlocked by completing the 3-hall Gauntlet at `[-200, 181, -200]` (`Altar of the Shell` bounding volume `x=-201, y=183, z=-143, dx=2, dy=3, dz=2`).
  - Automatically transitioned to `gamemode survival` when warping to the Wild Frontier (`!wild`) and returned to `gamemode adventure` when entering the Estate (`!estate`).

### 3.2 Chat Command Dispatch Table

| COMMAND_TOKEN | ALIASES | ACL_REQUIREMENT | RCON_EXECUTION_SEQUENCE |
| :--- | :--- | :--- | :--- |
| `!estate` | `!spawn`, `!hub`, `!home`, `!back` | `ANY` | `gamemode adventure <p>` (or `survival` if `OP`) -> `tp <p> -282 81 -348 -90 0` |
| `!gauntlet` | `!jq`, `!trial` | `ANY` | `gamemode adventure <p>` -> `tp <p> -200 181 -200 0 0` -> `clear <p>` -> `give <p> stone_sword 1` -> `give <p> shield 1` |
| `!wild` | `!frontier`, `!rtp`, `!survival` | `OP \|\| SURVIVOR` | `gamemode survival <p>` -> `spreadplayers -283 -350 750 2000 false <p>` -> `effect give <p> slow_falling 15 1 true` -> `effect give <p> resistance 15 4 true` |
| `!wiki` | `!repo`, `!github`, `!guide`, `!site`, `!web` | `ANY` | Emits clickable `tellraw` JSON link to `https://github.com/borocode/boro-realm` |
| `!help` | `!commands`, `!cmd`, `?` | `ANY` | Prints command reference table to player chat buffer |
| `!ping` | `ping`, `test`, `you there?` | `ANY` | Returns `[BORO BRIDGE] PONG! Bridge online.` |

---

## 04 // VOXEL_ARCHITECTURE_AND_ARTERIAL_GRAPH

Centered at `X=-296..-221, Y=59..98, Z=-375..-321`. Built to strict human-scale architectural cohesion: zero floating platforms, continuous weatherproof roofs, grounded foundations, and 3-block vertical headroom across all stairs and corridors.

### 4.1 Vertical Elevation Stack (Sovereign Grand Chalet)

| LEVEL_ID | Y_INTERVAL | X_Z_BOUNDS | STRUCTURAL_SPECIFICATION | VERTICAL_INTERCONNECT |
| :--- | :--- | :--- | :--- | :--- |
| `L5_ROOF_SPINE` | `Y=88..97` | `X=-296..-279, Z=-354..-340` | Continuous pitched dark-oak stair roof backed by solid planks; gable end walls (`X=-295`, `X=-280`); tinted-glass skylight (`Z=-347, Y=95`); twin stone-brick chimneys (`Y=96`) with lightning rods (`Y=97`). | Covers `L4_STARLIGHT_LOFT` |
| `L4_STARLIGHT_LOFT` | `Y=88..94` | `X=-289..-284, Z=-350..-343` | 3rd-floor timber mezzanine deck under cathedral skylight; oak balustrade, star-chart lectern (`-285, 89, -347`), cartography table, amethyst cluster. | Loft Stairs (`X=-285..-288, Z=-342, Y=85..88`) |
| `L3_PENTHOUSE` | `Y=84..87` | `X=-295..-277, Z=-353..-341` | 2nd-floor spruce slab deck (`Y=84`); master bed (`-294, 85, -348`), enchanting dais (`-284, 85, -350`), Sunrise Balcony (`X=-279..-277`) with overhead pergola (`Y=88`). | North Flight (`X=-288..-284, Z=-352, Y=80..84`) + Library Flight (`X=-291, Z=-355..-358`) |
| `L2_GROUND_HALL` | `Y=80..83` | `X=-295..-277, Z=-359..-340` | Grand Foyer (`-282, 81, -348`), unobstructed central hall (`Z=-347`), western campfire hearth (`-294, 80, -347`), tiled south kitchen (`Z=-344..-341`), Alexandrian Library (`Z=-359..-355`), Covered East Porch (`X=-279..-277`). | Double Entry Doors (`-280, 81, -347`) |
| `L1_WINE_CELLAR` | `Y=74..77` | `X=-290..-281, Z=-351..-342` | Subterranean vault (`Y=74` floor); spruce parquet floor, oak aging barrel racks, sommelier tasting bar (`-285, 75, -346`), brewing stands, lantern lighting. | South Cellar Flight (`X=-287..-282, Z=-342, Y=80..75`) |
| `L0_THERMAL_SPA` | `Y=68..71` | `X=-290..-281, Z=-351..-342` | Subterranean geothermal bath (`-284, 67, -347`), smooth quartz relaxation deck, subfloor campfire steam vent (`-283, 66, -347` under `iron_bars`), soul lanterns. | West Spa Flight (`X=-289, Z=-349..-343, Y=74..68`) |
| `LB_PIONEERS_CRYPT` | `Y=59..63` | `X=-290..-281, Z=-351..-340` | Deepest sanctuary (`Y=59` floor); stone-brick sarcophagus (`-285, 60, -346`), double reliquary chests (`-285, 61, -349`), chiseled bookshelves, soul fire braziers. | Crypt Flight (`X=-289, Z=-350..-340, Y=68..59`) |

### 4.2 Grounded Arterial Road Graph

| CORRIDOR_ID | COORDINATE_VECTOR | PAVING_COMPOSITION | CONNECTED_FACILITIES |
| :--- | :--- | :--- | :--- |
| `R1_EAST_AVENUE` | `X=-276..-242, Y=80..74, Z=-348..-346` | 3-wide `stone_bricks` + `cobblestone` with `stone_brick_wall` curbs & timber architraves | Chalet Front Porch (`X=-277`), North/South Forecourt Gardens (`X=-276`), Grand Mountain Entry Gate (`X=-258`, 3-block headroom `Y=75..77`), Bazaar Branch (`X=-252`), Barracks Branch (`X=-250`). |
| `R2_VALLEY_CAUSEWAY` | `X=-242..-221, Y=74..64, Z=-348..-346` | Stepped `stone_bricks` ramp -> `spruce_planks` lake pier on `cobblestone_wall` pilings | Connects East Avenue terminus (`X=-242, Y=74`) down to Reflection Lake shoreline (`Y=64`), East Lake Pier (`X=-230..-221`), boat mooring, and fishing barrel. |
| `R3_NORTH_MARINA` | `X=-275..-267, Y=78..74, Z=-349..-375` | Pier-supported `stone_brick_stairs` -> `spruce_planks` boardwalk -> `cobblestone` trail | Connects East Avenue (`Z=-349`) to Reflection Lake Marina Boardwalk (`Z=-363..-358`), Covered Gazebo (`Y=78`), and North Cliff Overlook (`Z=-375, Y=75`). |
| `R4_SW_INDUSTRIAL` | `X=-275..-305, Y=79..73, Z=-345..-316` | 3-wide `cobblestone` + `gravel` + `stone_bricks` stepped road | Connects East Avenue (`Z=-345`) to Royal Stables (`Z=-335`), Master Blacksmith Forge (`X=-291`), South Mine Railhead Depot (`X=-295, Z=-330..-321`), Apiary Promenade (`Z=-316`), and West Valley Overlook (`X=-305, Y=73`). |
| `R5_NE_SE_BRANCHES` | `X=-252..-250, Y=74, Z=-360..-335` | 3-wide `stone_bricks` + `cobblestone` lateral avenues | North branch (`X=-252, Z=-360..-349`) serves Market Bazaar stalls; South branch (`X=-250, Z=-345..-335`) serves Guard Barracks & Wolf Kennels. |
| `R6_WEST_AERODROME` | `X=-301..-298, Y=83..98, Z=-355..-343` | Cliffside `stone_brick_stairs` switchbacks + mid-mountain rest landing (`Y=91`) | Connects West Chalet Terrace (`Y=83`) to the `Y=98` High-Peak Elytra Aerodrome & Summit Arrival Arch (`-299, 102, -355`). |

---

## 05 // STRUCTURAL_COHESION_AUDIT_INVARIANTS (45/45)

Verified via [`master_50_phase_audit.py`](docs/estate.md) over RCON (`docker exec -i boro-minecraft rcon-cli`):

- **Weatherproofing & Roof Envelope (`01..06`):** Pitched dark-oak stair roof (`-287 91 -347`), tinted-glass skylight (`-287 95 -347`), solid west/east gable walls (`-295 89 -347`, `-280 89 -347`), main chimney smoke (`-295 96 -347`), kitchen flue smoke (`-285 95 -340`).
- **3-Block Headroom & Internal Circulation (`07..16`):** Grand Foyer air envelope (`-282 81 -347` to `-282 83 -347`), Great Hall unobstructed corridor (`-292 81 -348` to `-281 82 -346`), Hearth Parlor armchair (`-291 80 -349`), Kitchen granite floor & smoker (`-285 80 -342`, `-285 81 -341`), Library Scriptorium (`-289 81 -356`, `-290 85 -356`), North Penthouse stairs & headroom (`-288 80 -352`, `-284 84 -352`, `-286 83 -352` to `-286 85 -352`), Penthouse deck & bed (`-288 84 -347`, `-294 85 -348`), Sunrise Balcony pergola (`-278 88 -347`), Starlight Loft stairs, floor, and lectern (`-285 85 -342`, `-286 88 -347`, `-285 89 -347`), South Cellar stairs & headroom (`-287 80 -342`, `-282 75 -342`, `-285 78 -342` to `-285 80 -342`).
- **Subterranean Vaults (`17..20`):** Level 1 Wine Cellar floor & tasting bar (`-285 74 -347`, `-285 75 -346`), Level 2 Thermal Spa stairs, pool, and steam vent (`-289 73 -343`, `-284 67 -347`, `-283 67 -347`), Level 3 Crypt stairs, floor, and reliquary (`-289 67 -349`, `-289 59 -341`, `-285 59 -347`, `-285 61 -349`).
- **Arterial Roads & Outlying Infrastructure (`21..41`):** Covered East Porch (`-278 80 -347`, `-278 84 -347`), Forecourt Gardens (`-276 80 -351`, `-276 80 -343`), Grand East Avenue & Architraves (`-276 80 -347`, `-242 74 -347`, `-266 79 -347`), Entry Gate Battlements & 3-block clearance (`-258 75 -348` to `-258 77 -346`, `-258 79 -347`), East Valley Causeway & Pier (`-241 74 -347`, `-231 64 -347`, `-221 64 -347`), North Marina Bridge, Boardwalk, Gazebo & Overlook (`-275 78 -350`, `-272 74 -360`, `-270 74 -361`, `-270 78 -361`, `-272 75 -375`), SW Stables/Forge Road, Railhead Depot, West Overlook & Apiary (`-275 79 -344`, `-291 74 -334`, `-295 74 -329`, `-295 74 -321`, `-305 73 -334`, `-282 74 -316`), NE Bazaar & SE Barracks Promenades (`-252 74 -355`, `-250 74 -340`), West Cliffside Switchbacks & Summit Aerodrome (`-301 85 -344`, `-299 98 -353`, `-299 102 -355`).
- **Server Rules & Entity Population (`42..45`):** `minecraft:fire_spread_radius_around_player == 0`, Lightning Rod protection (`-295 97 -347`, `-299 103 -355`), Immortal Villager stewardship (`63` active entities), Tamed Wolf & Cat companions (`26` active entities).

---

## 06 // DOCUMENTATION_INDEX

- [`docs/architecture.md`](docs/architecture.md) — Container stack, networking, RCON IPC, and security invariants.
- [`docs/commands.md`](docs/commands.md) — `bridge.py` chat command parser, ACL permissions, and RCON execution payloads.
- [`docs/estate.md`](docs/estate.md) — Coordinate-level specification of the Alpine Sovereign Estate (`Y=59..98`) and arterial road graph.
- [`docs/gauntlet.md`](docs/gauntlet.md) — Automated 3-hall verification trial (`[-200, 181, -200]`) and `bridge.py` polling mechanics.
- [`docs/frontier.md`](docs/frontier.md) — Wild Frontier survival mechanics, spreadplayers parameters, and villager trade matrix.
