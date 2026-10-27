# DOC_01 // CONTAINER_STACK_AND_NETWORK_ARCHITECTURE

```text
MODULE       : INFRASTRUCTURE_AND_IPC
COMPOSE_ROOT : Projects/minecraft-server/docker-compose.yml
SERVER_CORE  : PaperMC 26.3 (itzg/minecraft-server:latest)
BRIDGE_CORE  : Python 3.12 (bridge/bridge.py)
```

---

## 1. TOPOLOGY_DIAGRAM

```text
                           [ CLIENT_INGRESS_AND_HTTPS_PORTAL ]
                                            |
         +----------------------------------+----------------------------------+
         | (HTTPS / TCP 443)                | (Java TCP 25565 SRV)             | (Bedrock UDP 18629)
         v                                  v                                  v
realm.boroghor.com                 play.boroghor.com                  play.boroghor.com:18629
         |                       (_minecraft._tcp SRV)                         |
         v                                  |                                  v
[Cloudflare Tunnel]                         v                         [Playit UDP Tunnel]
         |                            bore.pub:43991                  (signaling-ala.tun.ply.gg)
         v                                  |                                  |
  [ boro-web:80 ]                           v                                  |
  (Host Port 8767)                   [ boro-tunnel ]                           |
                                    (Bore Rust Client)                         |
                                            |                                  |
                                            +----------------+-----------------+
                                                             |
                                                             v
                                              [ boro-minecraft (Paper 26.3) ]
                                              - 25565/tcp (Java Protocol)
                                              - 19132/udp (Geyser Bedrock)
                                              - 25575/tcp (Local RCON)
                                                             ^
                                                             | RCON + /data/logs Volume
                                                             v
                                                     [ boro-bridge ]
                                              (Python 3.12 Autonomous Daemon)
```

---

## 2. CONTAINER_SPECIFICATIONS

All container images are pinned by immutable SHA256 digest in `docker-compose.yml` to prevent floating image drift.

### 2.1 `mc-crossplay` (`boro-minecraft`)
- **Image:** `itzg/minecraft-server:latest@sha256:3168dfa30c35fa1f99b8edde120f934a03ee780fcf905f450a1dbd9551d719f5`
- **Engine:** `PAPER` (`VERSION: "26.3"`)
- **JVM Memory:** `MEMORY: "3G"` with Aikar's GC flags (`USE_AIKAR_FLAGS: "true"`)
- **Server Properties:**
  - `MODE: "adventure"`
  - `DIFFICULTY: "normal"`
  - `MAX_PLAYERS: "20"`
  - `VIEW_DISTANCE: "10"`
  - `SIMULATION_DISTANCE: "8"`
  - `ONLINE_MODE: "true"` (Mojang session auth for Java + Floodgate `auth-type: floodgate` Xbox Live auth for Bedrock)
  - `ENFORCE_SECURE_PROFILE: "false"` (permits Bedrock players lacking Java RSA chat signatures to chat securely)
  - `BROADCAST_RCON_TO_OPS: "false"`
  - `ENABLE_RCON: "true"` (`RCON_PORT: 25575`)
- **Plugin Stack (`/data/plugins`):**
  - `Geyser-Spigot.jar` — Translates Bedrock UDP (`19132/udp`) packets to Java protocol.
  - `floodgate-spigot.jar` — Authenticates Bedrock Xbox Live sessions with `.` prefix (`e.g., .rohG_oroB`).
  - `ViaVersion.jar` & `ViaBackwards.jar` — Multi-version protocol compatibility.
  - `CoreProtect.jar` — Sub-tick block, chest, and entity transaction logging.

### 2.2 `tunnel` (`boro-tunnel`)
- **Image:** `alpine:3.20@sha256:d9e853e87e55526f6b2917df91a2115c36dd7c696a35be12163d44e6e2a4b6bc`
- **Network Mode:** `service:mc-crossplay` (shares network namespace directly with `boro-minecraft`)
- **Command Loop:**
  ```sh
  while true; do
    bore local 25565 --local-host mc-crossplay --to bore.pub --port 43991 || sleep 3
    sleep 2
  done
  ```

### 2.3 `bridge` (`boro-bridge`)
- **Image:** `python:3.12-alpine@sha256:b658b05ce9c4eb97ec6fc19424818a1f31993659c2b505a532f470e83807815d`
- **Entrypoint:** `python -u /app/bridge.py`
- **Mounts:**
  - `./bridge:/app:ro`
  - `./data/logs:/data/logs`
  - `./data/.rcon-cli.env:/data/.rcon-cli.env:ro`
- **Core Functions:**
  - Persistent TCP RCON socket client (`mc-crossplay:25575`) with automatic reconnect.
  - Non-blocking file tailer tracking inode rotation on `/data/logs/latest.log`.
  - Strictly anchored log parsing regexes (`^...$`) to eliminate log-injection / command spoofing.
  - Authoritative RCON NBT UUID evaluation for ACL verification (`data get entity <p> UUID`).
  - `1.0s` polling loop (`check_gauntlet()`) executing RCON selector checks for the Boro Gauntlet at `[-200, 181, -200]`.
  - Stateful persistence in `/data/logs/survivors.json` and `/data/logs/operator_inbox.json`.

### 2.4 `web` (`boro-web`)
- **Image:** `nginx:alpine@sha256:a3554c0d0b48177b1413c26e01f695334428020d9df01243fbae3e11cfd17417`
- **Port Binding:** `8767:80/tcp`
- **Mount:**
  - `../boro-realm:/usr/share/nginx/html:ro`
  - `./nginx.conf:/etc/nginx/conf.d/default.conf:ro`
- **Security Headers & Dotfile Denial:** Custom `nginx.conf` blocks all hidden dotfiles (`.git`, `.env`, `.rcon*`) returning `404 Not Found`, and serves hardened security headers (`X-Frame-Options: SAMEORIGIN`, `X-Content-Type-Options: nosniff`).
- **Ingress:** Exposed publicly via Cloudflare Tunnel at `https://realm.boroghor.com/`.

---

## 3. SECURITY_AND_STATE_INVARIANTS

1. **Gateway Privacy:** Host residential IP is never exposed in DNS records or repository artifacts. Java traffic transits `bore.pub:43991`; Bedrock traffic transits `signaling-ala.tun.ply.gg:18629`; HTTP traffic transits Cloudflare edge.
2. **Fire Spread Suppression:** Gamerule `minecraft:fire_spread_radius_around_player` is locked to `0` so decorative hearth campfires (`-294, 80, -347`) and chimney flues (`Y=95..96`) never ignite surrounding timber.
3. **Zone Gamemode Isolation:** Non-operator players inside the Alpine Sovereign Estate (`!estate`) are strictly locked to `gamemode adventure`. `gamemode survival` is only active after conquering the Gauntlet and warping to the Wild Frontier (`!wild`).
4. **RCON Sanitization & Cryptographic UUID Authentication:** Player names extracted from `/data/logs/latest.log` are strictly validated against `^\.?\w{1,16}$` before interpolation into RCON commands. Operator privilege evaluation completely bypasses spoofable chat strings by directly querying RCON NBT integer arrays (`data get entity <p> UUID`) against the authorized Mojang and Floodgate Xbox UUIDs.
5. **Container Image Digests:** Floating `:latest` and `:alpine` tags are strictly pinned by sha256 immutable digest across all 4 compose stack containers.
