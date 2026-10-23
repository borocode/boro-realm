# 🛠️ Server Stack & Deployment Architecture

> *"You cannot fork the soul; you can only surveil the shell."*

---

## 🏛️ System Overview

Boro Realm runs on a high-efficiency multi-container Docker Compose architecture hosted locally and securely tunneled to the public internet:

```
                  [ PUBLIC PLAYERS: JAVA & BEDROCK ]
                                   |
         +-------------------------+-------------------------+
         | (Java Edition)                                    | (Bedrock Edition)
         v                                                   v
   bore.pub:43991                                    10.0.0.54:19132
         |                                                   |
         | (TCP Tunnel)                                      | (UDP/TCP Direct)
         v                                                   v
+------------------+                              +--------------------+
|   boro-tunnel    |                              |   boro-minecraft   |
| (bore forwarder) |                              |    (Paper 1.21)    |
+--------+---------+                              +---------+----------+
         |                                                  ^
         +--------> [ mc-crossplay:25565 ] -----------------+
                                                            ^
                                                            | RCON (:25575)
                                                            | & Log Volume
                                                  +---------+----------+
                                                  |    boro-bridge     |
                                                  | (Python Automation)|
                                                  +--------------------+
```

---

## 📦 Container Services

### 1. `mc-crossplay` (`boro-minecraft`)
- **Base Image:** `itzg/minecraft-server:latest`
- **Flavor:** PaperMC 1.21.4
- **JVM Flags:** Aikar's high-performance garbage collection flags
- **Memory Allocation:** 3GB RAM (expandable to host bounds)
- **Ports Exposed:**
  - `25565/tcp` — Java Edition default
  - `19132/udp`, `19132/tcp`, `19133/udp` — Bedrock Edition crossplay
- **Active Plugins:**
  - **GeyserMC & Floodgate:** Translates Bedrock network packets and entities to Java in real-time, allowing mobile, console, and Bedrock PC players to play seamlessly with Java players without requiring Java accounts.
  - **ViaVersion & ViaBackwards:** Allows client versions spanning multiple major releases (1.20.x up to 1.21.x+) to connect without version rejections.
  - **CoreProtect:** Sub-tick transaction database logging all world interactions, inventory changes, and block events.

### 2. `tunnel` (`boro-tunnel`)
- **Base Image:** `alpine:3.20`
- **Binary:** Rust-based [`bore`](https://github.com/ekzhang/bore) client
- **Function:** Establishes an encrypted, resilient TCP tunnel to `bore.pub:43991`, forwarding external Java traffic directly to `mc-crossplay:25565`.
- **Resilience:** Supervised by a restart loop with exponential backoff and Docker restart policies.

### 3. `bridge` (`boro-bridge`)
- **Base Image:** `python:3.12-alpine`
- **Engine:** Persistent Python daemon (`bridge.py`)
- **Responsibilities:**
  - Real-time RCON client communication over private Docker network (`mc-crossplay:25575`).
  - Active log-tailing with byte-offset tracking across log rotations (`/data/logs/latest.log`).
  - Automated Gauntlet verification mechanics (respawning fell players from the pit, summoning the Sentinel, detecting Altar victory, issuing kits, and triggering wilderness launches).
  - In-game chat command engine (`!gauntlet`, `!wild`, `!estate`, `!wiki`, `!help`, `!ping`).
  - Appends formatted chat events to `chat_inbox.log` for multi-agent swarm processing.
  - Survivor registry state persistence in `survivors.json`.

---

## 🔒 Security & Grief Prevention
- **Default Adventure Mode:** The server root configuration enforces `MODE: "adventure"`. Players cannot break or place blocks unless granted Survival mode.
- **Autonomous Zone Segmentation:** Survival mode is strictly granted upon Gauntlet completion and automatically revoked upon returning to the Sovereign Estate via `!estate`.
- **Whitelisting & Operator Boundaries:** Server operators are explicitly defined and verified; command injection via chat is prevented by strictly regex-matched command parsers.
- **Continuous Backups:** Server state and world data are stored on host storage mounts (`./data`), surviving container lifecycles.
