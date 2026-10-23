# 📜 In-Game Commands Reference

The Boro Realm server runs a persistent bridge daemon that processes chat commands in real-time through RCON without requiring client-side mods or permission plugins.

---

## ⚡ Player Commands

### `!gauntlet`
- **Aliases:** `!jq`, `!trial`
- **Allowed Mode:** Any
- **Destination:** `[-200.0, 181.0, -200.0]` (The Boro Gauntlet Antechamber)
- **Gamemode on Warp:** `adventure`
- **Inventory Action:** Clears old trial gear and grants `1x Stone Sword` and `1x Shield`.
- **Purpose:** Warps the player to the beginning of the Verification Gauntlet. Players who fell into the pit or who wish to replay the parkour trial can use this at any time.

---

### `!wild`
- **Aliases:** `!frontier`, `!rtp`, `!survival`
- **Allowed Mode:** Verified Survivors & Server Operators
- **Destination:** Random location `750` to `2000` blocks away from the Sovereign Estate center `[-283, -350]`.
- **Gamemode on Warp:** `survival`
- **Description:** Launches the player into the untamed Wild Frontier where terrain modification, mining, gathering, hunting, crafting, and base-building are completely enabled.
- **Requirement:** Players who have not completed the Verification Gauntlet will receive an access denied message instructing them to run `!gauntlet`.

---

### `!estate`
- **Aliases:** `!spawn`, `!hub`, `!home`, `!back`
- **Allowed Mode:** Any
- **Destination:** `[-282.0, 81.0, -348.0]` (The Grand Foyer of the Sovereign Grand Chalet)
- **Gamemode on Warp:** `adventure` (Non-OPs) / `survival` (OPs)
- **Description:** Returns the player safely to the Alpine Sovereign Estate. In Adventure mode, players can freely explore all 50 phases and 60 enhancements, trade with the 63 immortal villagers, use crafting tables, enter the Nether Portal, or rest without risk of monster attacks or accidental griefing.

---

### `!wiki`
- **Aliases:** `!repo`, `!github`, `!guide`, `!site`, `!web`
- **Allowed Mode:** Any
- **Description:** Sends a JSON formatted chat message with a clickable hyperlink to the official Boro Realm repository: `https://github.com/borocode/boro-realm`.

---

### `!help`
- **Aliases:** `!commands`, `!cmd`, `?`
- **Allowed Mode:** Any
- **Description:** Prints the colored command reference table to the player's personal chat buffer.

---

### `!ping`
- **Aliases:** `ping`, `test`, `you there?`
- **Allowed Mode:** Any
- **Description:** Tests the bridge daemon's responsiveness and verifies whether the background AI supervisor is active.

---

## 🛠️ Contributor & Agent Guide: Adding New Commands

To register a new command on Boro Realm:
1. Open [`Projects/minecraft-server/bridge/bridge.py`](https://github.com/borocode/boro-realm).
2. Locate the `handle_line(line)` function and the `CHAT_RE` parser.
3. Add a new branch under `if c in (...):`.
4. Execute any RCON or custom logic via `rcon.cmd(...)`.
5. Update this `docs/commands.md` file and push to GitHub!
