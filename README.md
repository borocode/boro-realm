# ⚡ BORO REALM ⚡
> Sovereign Alpine Cross-Play Minecraft Universe — High-Altitude Architecture, Verification Gauntlet & Untamed Wild Frontier.

```
+---------------------------------------------------------------------------------------+
|  HOST: play.boroghor.com          |  ENGINE: PaperMC 1.21.4 (Java + Bedrock Cross-Play) |
|  STATUS: ONLINE 24/7 (AUTONOMOUS) |  BRIDGE: boro-bridge container daemon (zero LLM)  |
+---------------------------------------------------------------------------------------+
```

---

## 🌐 Network & Quick Connect

Connect with any modern client (PC, Mobile, Nintendo Switch, Xbox, PlayStation):

| Client Protocol | Connect Host / Address | Port | Protocol Engine | Notes |
| :--- | :--- | :--- | :--- | :--- |
| **Java Edition** | `play.boroghor.com` | `25565` *(Default)* | PaperMC / SRV | Cloudflare SRV auto-routes. No port entry required. |
| **Bedrock Edition** | `play.boroghor.com` | `18629` | Geyser / Floodgate | Windows 10/11, iOS, Android, Switch, Consoles. |
| **Java Direct Fallback** | `bore.pub` | `43991` | Bore TCP Tunnel | Use if DNS client fails to query SRV records. |
| **Bedrock Direct Fallback** | `signaling-ala.tun.ply.gg` | `18629` | Playit Tunnel | Direct Playit global tunnel routing. |

> **Zero Client Downloads:** Pure vanilla network protocol. No client mods, Forge, Fabric, or custom resource packs needed.

---

## ⚡ Player Lifecycle State Machine

```
                                [ Wanderer First Connect ]
                                            │
                                            ▼
                      [ Phase 1: Boro Gauntlet (Adventure Mode) ]
                                   [-200, 181, -200]
                                            │
                 ┌──────────────────────────┼──────────────────────────┐
                 │                          │                          │
                 ▼                          ▼                          ▼
          [ Hall I: Abyss ]        [ Hall II: Colosseum ]      [ Hall III: Altar ]
          Precision parkour         Vanquish Sentinel           Touch Obsidian Dais
                 │                          │                          │
                 └──────────────────────────┼──────────────────────────┘
                                            │
                                            ▼
                              [ ⚡ Celestial Lightning ⚡ ]
                            Server-wide victory broadcast
                            Grant Survivor Starter Kit
                                            │
                                            ▼
                       [ Phase 2: Wild Frontier (Survival Mode) ]
                          Launched 750–2,000 blocks into wild
                          Build, mine, quarry, trade & claim
                                    │               ▲
                                    │ (!estate)     │ (!wild)
                                    ▼               │
                      [ Phase 3: Alpine Sovereign Estate (Safe Hub) ]
                                   [-282, 81, -348]
                          63 master villagers, forge, library,
                          weatherproof chalet, grief-proof
```

---

## 🏰 Architectural Structure: The Alpine Sovereign Estate

Built at mountain coordinates `[-296..-221, Y=59..98, -375..-321]`, the estate adheres to grounded alpine cohesion:

- **100% Weatherproof A-Frame Chalet:** Continuous pitched roof (`Y=88..95`) backed by solid planks, gable end walls (`X=-295` & `X=-280`), cathedral tinted-glass skylight, and twin stone-brick chimneys with live hearth smoke (`Y=96`).
- **Headroom & Circulation Invariant:** Full 3-block clear vertical headroom across all hallways, stairs, and doors. Zero floating 6×6 platforms or copy-pasted templates.
- **Walkable Vertical Staircase Network:**
  - *North Penthouse Flight:* `Y=80` (Ground Floor) $\rightarrow$ `Y=84` (Penthouse).
  - *Starlight Loft Flight:* `Y=85` (Penthouse) $\rightarrow$ `Y=88` (Starlight Mezzanine Loft).
  - *South Cellar Flight:* `Y=80` (Ground Floor) $\rightarrow$ `Y=74` (Level 1 Wine Cellar).
  - *Subterranean Spa Flight:* `Y=74` (Level 1) $\rightarrow$ `Y=68` (Level 2 Thermal Spa Lounge).
  - *Subterranean Crypt Flight:* `Y=68` (Level 2) $\rightarrow$ `Y=59` (Level 3 Pioneers Crypt).
- **Connected Arterial Roads:**
  - *Grand East Avenue (`X=-276..-242, Y=80..74`):* 3-block-wide stone-brick & cobblestone stepped road through the fortified Mountain Entry Gate.
  - *East Valley Shoreline Causeway & Pier (`X=-242..-221, Y=74..64`):* Paved road and timber lake pier with boat moorings.
  - *North Lakeside Stairway & Marina (`X=-275..-267, Y=78..74`):* Pier-supported bridge, timber boardwalk, and sheltered fishing gazebo.
  - *South-West Stables, Forge & Railhead (`X=-275..-305, Y=79..73`):* Paved road connecting the stables, master forge, deep mine railhead, and apiary.
  - *West Mountain Switchback (`X=-301..-298, Y=83..98`):* Cliffside stairs leading to the high-peak Elytra Aerodrome summit.

---

## 📜 In-Game Chat Bridge Commands

Handled autonomously in-container by `boro-bridge` daemon:

| Command | Aliases | Target Gamemode | Description |
| :--- | :--- | :--- | :--- |
| `!gauntlet` | `!jq`, `!trial` | Adventure | Warp to the Gauntlet starting gates (`-200, 181, -200`). |
| `!wild` | `!frontier`, `!rtp` | Survival | Spread-teleport verified survivors into random wilderness. |
| `!estate` | `!spawn`, `!hub`, `!home` | Adventure | Warp to Alpine Sovereign Estate Grand Foyer (`-282, 81, -348`). |
| `!wiki` | `!repo`, `!guide` | Any | Prints documentation & web portal link. |
| `!help` | `!cmd`, `?` | Any | Outputs command palette directly in chat. |
| `!ping` | `ping`, `test` | Any | Verifies bridge latency and responsiveness. |

---

## ⚙️ Container Infrastructure

```
                                [ Public Internet ]
                                         │
                    ┌────────────────────┴────────────────────┐
                    │                                         │
                    ▼                                         ▼
            [ play.boroghor.com ]                     [ play.boroghor.com ]
            Java SRV -> bore.pub:43991                Bedrock -> ply.gg:18629
                    │                                         │
                    ▼                                         ▼
             [ bore-tunnel ]                            [ playit ]
            Forwarder Container                      Direct Tunnel Node
                    │                                         │
                    └────────────────────┬────────────────────┘
                                         │
                                         ▼
                             [ mc-crossplay Container ]
                             PaperMC 1.21.4 (Port 25565 / UDP 18629)
                             ├── GeyserMC (Bedrock Protocol Translation)
                             ├── Floodgate (Authentication Bypass)
                             ├── CoreProtect (Sub-tick Block Logging)
                             └── ViaVersion / ViaBackwards
                                         ▲
                                         │ Localhost RCON
                                         ▼
                              [ boro-bridge Container ]
                             Python 3.12 Autonomous Daemon
                             ├── Tail /data/logs/latest.log
                             ├── Gauntlet Completion Detection
                             ├── Zero-LLM In-Game Chat Parser
                             └── Async Operator Queue (/data/logs/operator_inbox.json)
```

---

## 🛡️ Sovereign Invariants
1. **Fire Propagation:** Namespaced rule `minecraft:fire_spread_radius_around_player` permanently locked to `0`.
2. **IP Privacy:** Residential gateway `24.98.32.114` is never broadcast or bound to DNS. All ingress routes through Cloudflare SRV and encrypted TCP tunnels.
3. **Estate Integrity:** Estate zone is permanently protected in Adventure Mode. Block destruction is restricted to wilderness sectors.
4. **Autonomous AI Pair-Programming:** Asynchronous in-game operator commands from sovereign players (`steveXBT`, `.rohG_oroB`) are queued to `operator_inbox.json` and executed cyclically.

---

*Maintained autonomously by Antigravity, Hermes, and the OpenClaw Swarm.*
