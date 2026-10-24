# DOC_04 // WILD_FRONTIER_SURVIVAL_SPECIFICATION

```text
MODULE          : SURVIVAL_FRONTIER_SECTOR
ORIGIN_CENTER   : X=-283, Z=-350
INNER_RADIUS    : 750 blocks
OUTER_RADIUS    : 2000 blocks
GAMEMODE_POLICY : survival (CoreProtect audited)
```

---

## 1. SECTOR_PARAMETERS

The **Wild Frontier** comprises all procedural world chunks outside the `750`-block radius centered on `[-283, -350]`.

- **Gamemode:** `survival` (full block harvesting, placement, crafting, mining, redstone, and dimension travel).
- **Entry Mechanism:** Triggered automatically upon completing the Boro Gauntlet at `[-201, 183, -143]` or manually at any time by verified survivors via `!wild` (`!frontier`, `!rtp`).
- **Teleport Command:**
  ```mcfunction
  spreadplayers -283 -350 750 2000 false <player>
  effect give <player> minecraft:slow_falling 15 1 true
  effect give <player> minecraft:resistance 15 4 true
  ```

---

## 2. SURVIVOR_STARTER_KIT

Issued once upon initial Gauntlet verification (`survivors.json` registration):

| ITEM_ID | QUANTITY | FUNCTIONAL_ROLE |
| :--- | :--- | :--- |
| `minecraft:bread` | `16` | Initial saturation and stamina recovery |
| `minecraft:stone_pickaxe` | `1` | Immediate stone/coal extraction |
| `minecraft:stone_axe` | `1` | Timber harvesting |
| `minecraft:torch` | `16` | Subterranean illumination |
| `minecraft:water_bucket` | `1` | Fall mitigation, hydration, and lava conversion |

---

## 3. RETURN_AND_TRADE_LOOP

1. **Return to Safe Hub:** Executing `!estate` (`!spawn`, `!hub`) from anywhere in the Wild Frontier teleports the player to `[-282.0, 81.0, -348.0]` and sets `gamemode adventure`.
2. **Villager Trading:** While in `adventure` mode at the Alpine Sovereign Estate, players can trade with all `63` master villagers (Librarians, Armorsmiths, Weaponsmiths, Toolsmiths, Masons, Farmers) and access crafting/enchanting stations (`-284, 85, -350`).
3. **Re-Entry to Frontier:** Executing `!wild` restores `gamemode survival` and deploys the player back into the `750..2000` block frontier ring.
4. **Block Audit Trail:** All block placements, breakages, and container transactions in the Wild Frontier are logged to SQLite by **CoreProtect** (`/co inspect`, `/co rollback`).
