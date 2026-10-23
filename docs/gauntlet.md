# ⚡ The Boro Gauntlet: The Verification Trial

> *"Only those with steady footing and resolute blade shall inherit the Wild Frontier."*

---

## 🏛️ Overview
The **Boro Gauntlet** is the automated gatekeeper of Boro Realm. Floating high above the bedrock at coordinates `X=-200, Y=181, Z=-200`, this colossal stone sanctuary challenges every newly arrived wanderer to demonstrate parkour agility and martial discipline before receiving building and survival permissions.

```
[ ENTRY SANCTUARY ] ----> [ HALL I: EMBER ABYSS ] ----> [ HALL II: IRON COLOSSEUM ] ----> [ HALL III: ALTAR ]
  (-200, 181, -200)           Parkour Chasm              Sentinel of the Shell             Lightning Strike
```

---

## ⚔️ The Three Trials

### Hall I: The Ember Abyss (`Z = -200` to `-180`)
- **Challenge:** Precision parkour across narrow basalt and deepslate pillars hovering over a magma pit.
- **Fail Detection:** If a wanderer misses a leap and drops below $Y=175$, an automated pressure sensor triggers an immediate respawn at the hall entry:
  ```mcfunction
  execute as @a[x=-207,y=175,z=-193,dx=14,dy=3,dz=21] at @s run tp @s -200 181 -200 0 0
  ```
- **Tip:** Take your time; sprint-jumping is only required on the final double-gap!

### Hall II: The Iron Colosseum (`Z = -180` to `-160`)
- **Challenge:** A fortified combat ring presided over by the ancient **Sentinel of the Shell** — an armored archer wielding enchanted iron weaponry and chainmail.
- **Trial Equipment:** All challengers are equipped with a Stone Sword and Shield upon entry.
- **Mechanic:** The bridge daemon monitors the chamber and summons the Sentinel when a challenger enters:
  ```mcfunction
  summon minecraft:skeleton -200 181 -161 {
    HandItems:[{id:"minecraft:iron_sword",count:1}],
    ArmorItems:[{},{},{id:"minecraft:chainmail_chestplate",count:1},{id:"minecraft:iron_helmet",count:1}],
    CustomName:'"⚡ Sentinel of the Shell ⚡"',
    CustomNameVisible:1b,
    PersistenceRequired:1b
  }
  ```
- **Tip:** Keep your shield raised to deflect arrows, then close the distance and strike between volleys!

### Hall III: The Altar of the Shell (`Z = -160` to `-140`)
- **Coordinate:** `[-201, 183, -143]`
- **Challenge:** Ascent up the ceremonial crying obsidian dais to the glowing altar beacon.
- **Completion Ceremony:**
  - Celestial lightning bolt strikes the altar spire!
  - Toast challenge completion audio plays for all online players.
  - Server-wide banner broadcast:
    ```
    ⚡ [BORO SANCTUM] <Player> conquered the Gauntlet and unlocked SURVIVAL in the Wild Frontier! ⚡
    ```
  - Trial weapons are cleared.
  - The **Survivor Starter Kit** is granted:
    - 16x Bread
    - 1x Stone Pickaxe
    - 1x Stone Axe
    - 16x Torches
    - 1x Water Bucket
  - Player gamemode transitions to `survival`.
  - The player is catapulted into the wilderness via `/spreadplayers -283 -350 750 2000 false`!
  - The player's identity is permanently recorded in `survivors.json`.
