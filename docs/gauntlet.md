# DOC_05 // BORO_GAUNTLET_VERIFICATION_SPECIFICATION

```text
MODULE        : GAUNTLET_TRIAL_FSM
ENTRY_ANCHOR  : [-200.0, 181.0, -200.0, yaw=0.0, pitch=0.0]
ALTAR_ANCHOR  : [-201.0, 183.0, -143.0, dx=2, dy=3, dz=2]
POLL_INTERVAL : 2.0 seconds (bridge.py :: check_gauntlet)
STATE_STORE   : /data/logs/survivors.json
```

---

## 1. STATE_MACHINE_OVERVIEW

```text
[ UNVERIFIED_GUEST ]
         |
         | (!gauntlet)
         v
[ HALL_01 : EMBER_ABYSS ]        (Z = -200 .. -180 | Parkour over Y=175 pit)
         |
         +---> [ Y <= 178 PIT_FALL ] ---> tp @s -200 181 -200 0 0 (Retry)
         |
         v
[ HALL_02 : IRON_COLOSSEUM ]     (Z = -180 .. -160 | Sentinel Boss Trigger)
         |
         +---> Summons "Sentinel of the Shell" at [-200, 181, -161]
         |
         v
[ HALL_03 : ALTAR_OF_THE_SHELL ] (X=-201..199, Y=183..186, Z=-143..-141)
         |
         v
[ VERIFIED_SURVIVOR ]            (Recorded in survivors.json -> Launched to !wild)
```

---

## 2. HALL_BY_HALL_MECHANICS

### 2.1 Hall I: The Ember Abyss (`Z = -200..-180`)
- **Geometry:** Suspended basalt and deepslate parkour pillars spanning `Z=-200` to `Z=-180` at `Y=181`.
- **Pit Fall Recovery (`check_gauntlet` step 1):** Every `2.0s`, `bridge.py` checks the pit bounding box (`x=-207, y=175, z=-193, dx=14, dy=3, dz=21`) and teleports fallen players back to the entry platform:
  ```mcfunction
  execute as @a[x=-207,y=175,z=-193,dx=14,dy=3,dz=21] at @s run tp @s -200 181 -200 0 0
  ```

### 2.2 Hall II: The Iron Colosseum (`Z = -180..-160`)
- **Geometry:** Enclosed combat arena spanning `x=-206..-194, y=180..186, z=-172..-156`.
- **Boss Spawn Trigger (`check_gauntlet` step 2):** When a player enters the arena volume and no Sentinel is present within `18` blocks, `bridge.py` spawns the boss:
  ```mcfunction
  summon minecraft:skeleton -200 181 -161 {
    HandItems:[{id:"minecraft:iron_sword",count:1}],
    ArmorItems:[{},{},{id:"minecraft:chainmail_chestplate",count:1},{id:"minecraft:iron_helmet",count:1}],
    CustomName:'"⚡ Sentinel of the Shell ⚡"',
    CustomNameVisible:1b,
    PersistenceRequired:1b
  }
  ```

### 2.3 Hall III: Altar of the Shell (`Z = -160..-140`)
- **Detection Volume (`check_gauntlet` step 3):**
  ```mcfunction
  execute if entity @a[x=-201,y=183,z=-143,dx=2,dy=3,dz=2,gamemode=adventure]
  ```
- **Verification Sequence (`grant_survivor(player)`):**
  1. Adds `player` to `survivors` set and persists `/data/logs/survivors.json`.
  2. Strikes lightning at the altar (`execute at <p> run summon minecraft:lightning_bolt ~ ~ ~`).
  3. Plays `minecraft:ui.toast.challenge_complete` to `@a`.
  4. Clears trial gear (`clear <p>`) and grants the Survivor Starter Kit (`16x bread`, `1x stone_pickaxe`, `1x stone_axe`, `16x torch`, `1x water_bucket`).
  5. Sets `gamemode survival <p>`.
  6. Executes `spreadplayers -283 -350 750 2000 false <p>` with `15s` of `slow_falling` and `resistance`.
