# DOC_03 // ALPINE_SOVEREIGN_ESTATE_SPECIFICATION

```text
MODULE         : VOXEL_ARCHITECTURE_AND_COHESION
BOUNDING_BOX   : X=[-305..-221], Y=[59..103], Z=[-375..-316]
SPAWN_ANCHOR   : [-282.0, 81.0, -348.0, yaw=-90.0, pitch=0.0]
AUDIT_COVERAGE : 45/45 RCON BLOCK & ENTITY INVARIANTS
```

---

## 1. ARCHITECTURAL_INVARIANTS

1. **Zero Floating Artifacts:** All buildings, terraces, bridges, and piers are anchored down to natural terrain or stone-brick/cobblestone pilings.
2. **Weatherproof Envelope:** The Sovereign Grand Chalet (`X=-296..-279, Z=-354..-340`) is sealed by a continuous pitched dark-oak stair roof (`Y=88..95`) backed by solid `dark_oak_planks`, closed gable walls (`X=-295`, `X=-280`), and a central `tinted_glass` cathedral skylight (`Z=-347, Y=95`).
3. **3-Block Headroom Invariant:** Every internal hallway, doorway, staircase flight, and exterior gate maintains at least 3 blocks of unobstructed vertical air clearance (`Y..Y+2`).
4. **Continuous Arterial Connectivity:** Every outlying landmark is connected back to the Chalet East Porch (`X=-277, Y=80, Z=-347`) via paved 3-block-wide stone-brick, cobblestone, or timber roads.

---

## 2. VERTICAL_FLOOR_SCHEDULE (SOVEREIGN GRAND CHALET)

| LEVEL_CODE | ELEVATION | COORDINATE_EXTENT | PRIMARY_ROOMS_AND_FIXTURES | STAIR_ACCESS |
| :--- | :--- | :--- | :--- | :--- |
| `ROOF_CROWN` | `Y=88..97` | `X=-296..-279, Z=-354..-340` | Pitched `dark_oak_stairs` roof, `tinted_glass` skylight (`Z=-347, Y=95`), Main Western Chimney (`-295, 96, -347`), South Kitchen Flue (`-285, 95, -340`), Lightning Rods (`Y=97`). | Covers `L4_LOFT` |
| `L4_LOFT` | `Y=88..94` | `X=-289..-284, Z=-350..-343` | **Starlight Observatory Mezzanine Loft:** `spruce_slab[type=top]` deck (`Y=88`), `oak_fence` railing, Star-Chart Lectern (`-285, 89, -347`), Cartography Table (`-287, 89, -349`), Amethyst Cluster (`-287, 90, -345`). | Loft Flight (`X=-285..-288, Z=-342, Y=85..88`) |
| `L3_PENTHOUSE` | `Y=84..87` | `X=-295..-277, Z=-353..-341` | **2nd-Floor Penthouse Suite:** Master Canopy Bed (`-294, 85, -348`), Enchanting Dais (`-284, 85, -350`), Library Upper Gallery (`Z=-358..-355`), Sunrise Balcony (`X=-279..-277`) with Pergola (`Y=88`). | North Flight (`X=-288..-284, Z=-352, Y=80..84`) |
| `L2_MAIN_FLOOR` | `Y=80..83` | `X=-295..-277, Z=-359..-340` | **Ground Floor:** Grand Foyer & Registry Console (`-282, 81, -350`), Unobstructed Great Hall (`Z=-347`), Inglenook Hearth & Parlor (`-294, 80, -347`), Tiled South Kitchen (`-285, 80, -342`), Alexandrian Library (`-289, 81, -356`), Covered East Porch (`-278, 80, -347`). | Front Double Doors (`-280, 81, -347`) |
| `L1_CELLAR` | `Y=74..77` | `X=-290..-281, Z=-351..-342` | **Vintage Wine Cellar:** `spruce_planks` floor (`Y=74`), Dual Oak Barrel Racks (`Z=-350`, `Z=-343`), Sommelier Tasting Counter (`-285, 75, -346`), Brewing Stands. | South Cellar Flight (`X=-287..-282, Z=-342, Y=80..75`) |
| `L0_SPA` | `Y=68..71` | `X=-290..-281, Z=-351..-342` | **Subterranean Thermal Spa:** Geothermal Hot Spring Pool (`-284, 67, -347`), Smooth Quartz Deck (`-283, 68, -349`), Subfloor Campfire Steam Grate (`-283, 67, -347`), Soul Lanterns. | West Spa Flight (`X=-289, Z=-349..-343, Y=74..68`) |
| `LB_CRYPT` | `Y=59..63` | `X=-290..-281, Z=-351..-340` | **Pioneers Crypt & Reliquary:** `deepslate_tiles` floor (`Y=59`), Ancestral Sarcophagus (`-285, 60, -346`), Reliquary Chests (`-285, 61, -349`), Chiseled Bookshelves (`-285, 61, -350`), Soul Fire Braziers. | Crypt Flight (`X=-289, Z=-350..-340, Y=68..59`) |

---

## 3. ARTERIAL_ROAD_AND_INFRASTRUCTURE_GRAPH

1. **Grand East Avenue (`X=-276..-242, Y=80..74, Z=-348..-346`):**
   - 3-block-wide stepped stone-brick and cobblestone avenue with `stone_brick_wall` curbs, overhead spruce timber architraves (`X=-266`, `X=-248`), and hanging lanterns.
   - Passes through the fortified **Grand Mountain Entry Gate (`X=-259..-257, Y=74..79`)** with full 3-block clearance (`Y=75..77`).
2. **East Valley Causeway & Shoreline Pier (`X=-242..-221, Y=74..64, Z=-348..-346`):**
   - Stepped stone-brick causeway descending from `X=-242, Y=74` to the eastern lake shoreline at `X=-231, Y=64`, terminating in a 3-block-wide spruce timber pier (`X=-230..-221, Y=64`) with mooring posts and fishing supplies.
3. **North Lakeside Stairway, Marina Boardwalk & Cliff Overlook (`X=-275..-267, Y=78..74, Z=-349..-375`):**
   - Pier-supported stairway linking the East Avenue to Reflection Lake (`Y=74`), connecting to the **Marina Boardwalk & Harbor Gazebo (`X=-274..-267, Z=-363..-358`)** and continuing north along the paved lakeside trail to the **North Cliff Overlook (`-272, 75, -375`)**.
4. **South-West Industrial Road, Stables, Forge, Railhead & Apiary (`X=-275..-305, Y=79..73, Z=-345..-316`):**
   - Continuous 3-block-wide road serving the **Royal Alpine Stables (`Z=-335`)**, **Master Blacksmith Forge (`-291, 74, -334`)**, **South Mine Railhead Depot (`X=-295, Z=-330..-321, Y=74`)**, **West Valley Overlook (`-305, 73, -334`)**, and **South Apiary Meadow (`-282, 74, -316`)**.
5. **North-East Bazaar & South-East Barracks Branches (`X=-252..-250, Y=74, Z=-360..-335`):**
   - Lateral stone-brick avenues branching off the Grand East Avenue at `X=-252` (Market Bazaar) and `X=-250` (Guard Barracks & Kennels).
6. **West Cliffside Switchback & High-Peak Elytra Aerodrome (`X=-301..-298, Y=83..103, Z=-355..-343`):**
   - Stepped mountain switchback trail ascending from the west chalet terrace (`Y=83`) past a mid-mountain rest landing (`Y=91`) to the `Y=98` **Elytra Aerodrome Launchpad** and **Summit Arrival Arch (`-299, 102, -355`)**.

---

## 4. ENTITY_POPULATION_SPECIFICATION

- **Immortal Master Villagers (`Count: 63`):** Tagged with `Invulnerable:1b, PersistenceRequired:1b`, distributed across the Library, Forge, Stables, Foyer, Bazaar, Marina, and Loft.
- **Tamed Companion Animals (`Count: 26`):** Guard wolves (`Sitting:1b, CollarColor:14b, Invulnerable:1b`) and domestic cats stationed across the Chalet hearths, Stables, and Entry Gate.
