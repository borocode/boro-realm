# DOC_02 // BRIDGE_IPC_AND_CHAT_COMMANDS

```text
MODULE      : BRIDGE_COMMAND_ENGINE
SOURCE_FILE : Projects/minecraft-server/bridge/bridge.py
LOG_SOURCE  : /data/logs/latest.log
RCON_TARGET : mc-crossplay:25575
```

---

## 1. COMMAND_DISPATCH_SPECIFICATION

`bridge.py` parses every chat event matching:
```python
CHAT_RE = re.compile(r"\[Async Chat Thread[^\]]*\]/INFO\]: (?:\[Not Secure\] )?<([^>]+)> (.*)")
```

### 1.1 `!estate`
- **Aliases:** `!spawn`, `!hub`, `!home`, `!back`
- **Access Control:** `ANY`
- **Target Coordinates:** `[-282.0, 81.0, -348.0, yaw=-90, pitch=0]` (Alpine Sovereign Estate Grand Foyer)
- **RCON Execution:**
  ```mcfunction
  gamemode adventure <player>   # or 'gamemode survival <player>' if player in OPS
  tp <player> -282 81 -348 -90 0
  ```

### 1.2 `!gauntlet`
- **Aliases:** `!jq`, `!trial`
- **Access Control:** `ANY`
- **Target Coordinates:** `[-200.0, 181.0, -200.0, yaw=0, pitch=0]` (Gauntlet Entry Sanctuary)
- **RCON Execution:**
  ```mcfunction
  gamemode adventure <player>
  tp <player> -200 181 -200 0 0
  clear <player>
  give <player> minecraft:stone_sword 1
  give <player> minecraft:shield 1
  ```

### 1.3 `!wild`
- **Aliases:** `!frontier`, `!rtp`, `!survival`
- **Access Control:** `player in OPS or player in survivors.json`
- **Target Coordinates:** Random surface coordinate `750..2000` blocks from `[-283, -350]`
- **RCON Execution (Authorized):**
  ```mcfunction
  gamemode survival <player>
  spreadplayers -283 -350 750 2000 false <player>
  effect give <player> minecraft:slow_falling 15 1 true
  effect give <player> minecraft:resistance 15 4 true
  ```
- **Rejection Action (Unauthorized):** Sends red `tellraw` warning prompting the player to complete `!gauntlet` first.

### 1.4 `!wiki`
- **Aliases:** `!repo`, `!github`, `!guide`, `!site`, `!web`
- **Access Control:** `ANY`
- **RCON Execution:** Emits a clickable `open_url` JSON `tellraw` linking to `https://github.com/borocode/boro-realm`.

### 1.5 `!help`
- **Aliases:** `!commands`, `!cmd`, `?`
- **Access Control:** `ANY`
- **RCON Execution:** Sends formatted command listing to the invoking player via `tellraw`.

### 1.6 `!ping`
- **Aliases:** `ping`, `test`, `you there?`
- **Access Control:** `ANY`
- **RCON Execution:** Sends `[BORO BRIDGE] PONG! Bridge online.` via `tellraw`.

---

## 2. OPERATOR_ASYNC_INBOX_PROTOCOL

When a sovereign operator (`OPS = {"steveXBT", ".rohG_oroB", "rohG_oroB"}`) sends a free-form chat message that does not match a built-in `!` command:

1. `bridge.py` appends an entry to `/data/logs/operator_inbox.json`:
   ```json
   {
     "timestamp": 1761336298.12,
     "player": "steveXBT",
     "message": "<raw message string>",
     "processed": false
   }
   ```
2. `bridge.py` immediately sends an in-game RCON confirmation:
   ```text
   [BORO BRIDGE] Operator message logged to AI inbox.
   ```
3. The autonomous maintenance loop reads `/data/logs/operator_inbox.json`, executes any requested world or server actions via RCON, replies directly in-game via `tellraw`, and updates `"processed": true`.
