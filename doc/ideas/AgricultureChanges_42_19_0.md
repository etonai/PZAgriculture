# Agriculture Structural Changes: 42.19.0

**Date**: 2026-06-13  
**Mod files**:
- `mymods/PseudoAgriculture/common/media/lua/server/PseudoGPTFarming.lua`
- `mymods/PseudoAgriculture/42/media/lua/server/RealisticKentuckyFarming.lua`

**Vanilla files (current)**:
- `media/lua/server/Farming/farming_vegetableconf.lua`
- `media/lua/server/Farming/farming_vegetableconf_vegetables.lua`
- `media/lua/server/Farming/farming_vegetableconf_herbs.lua`

---

## 1. `seasonRecipe` String Format Changed (Breaking)

The mod sets `seasonRecipe` using Title Case strings. The engine now expects a namespaced lowercase format. Every crop in the mod is affected.

**Mod format**: `"Barley Growing Season"`  
**Vanilla format**: `"base:barley growing season"`

This breaks the link between the mod's crop props and their corresponding season recipes. When a player examines a crop or the engine looks up the season recipe by ID, it will find no match.

**All affected crops in the mod**:

| Mod `seasonRecipe` | Required format |
|--------------------|-----------------|
| `"Barley Growing Season"` | `"base:barley growing season"` |
| `"Bell Pepper Growing Season"` | `"base:bell pepper growing season"` |
| `"Broccoli Growing Season"` | `"base:broccoli growing season"` |
| `"Cabbage Growing Season"` | `"base:cabbage growing season"` |
| `"Carrot Growing Season"` | `"base:carrot growing season"` |
| `"Cauliflower Growing Season"` | `"base:cauliflower growing season"` |
| `"Corn Growing Season"` | `"base:corn growing season"` |
| `"Cucumber Growing Season"` | `"base:cucumber growing season"` |
| `"Flax Growing Season"` | `"base:flax growing season"` |
| `"Garlic Growing Season"` | `"base:garlic growing season"` |
| `"Green Pea Growing Season"` | `"base:green pea growing season"` |
| `"Hemp Growing Season"` | `"base:hemp growing season"` |
| `"Hops Growing Season"` | `"base:hops growing season"` |
| `"Kale Growing Season"` | `"base:kale growing season"` |
| `"Lettuce Growing Season"` | `"base:lettuce growing season"` |
| `"Onion Growing Season"` | `"base:onion growing season"` |
| `"Potato Growing Season"` | `"base:potato growing season"` |
| `"Pumpkin Growing Season"` | `"base:pumpkin growing season"` |
| `"Radish Growing Season"` | `"base:radish growing season"` |
| `"Rye Growing Season"` | `"base:rye growing season"` |
| `"Soybean Growing Season"` | `"base:soybean growing season"` |
| `"Spinach Growing Season"` | `"base:spinach growing season"` |
| `"Strawberry Growing Season"` | `"base:strawberry growing season"` |
| `"Sugar Beet Growing Season"` | `"base:sugar beet growing season"` |
| `"Sunflower Growing Season"` | `"base:sunflower growing season"` |
| `"Sweet Potato Growing Season"` | `"base:sweet potato growing season"` |
| `"Tobacco Growing Season"` | `"base:tobacco growing season"` |
| `"Tomato Growing Season"` | `"base:tomato growing season"` |
| `"Turnip Growing Season"` | `"base:turnip growing season"` |
| `"Watermelon Growing Season"` | `"base:watermelon growing season"` |
| `"Wheat Growing Season"` | `"base:wheat growing season"` |
| `"Zucchini Growing Season"` | `"base:zucchini growing season"` |

---

## 2. New `badMonthHardyLevel` Property (Partial Impact)

The engine now reads a `badMonthHardyLevel` property on crop props to gate planting in bad months behind a farming skill level. Two vanilla crops define it:

- `Carrots`: `badMonthHardyLevel = 4`
- `Kale`: `badMonthHardyLevel = 3`

The mod overrides both of these crops and does not set `badMonthHardyLevel`. The engine behavior for a missing property is unknown without testing — it may silently default to 0 (no gate) or log an error. Either way the mod's intended bad-month behavior for these two crops may not match what the engine enforces.

---

## 3. Sandbox API Change (Engine-Level, Does Not Break Mod Files Directly)

`farming_vegetableconf.lua` replaced the old `SandboxVars.Farming` integer check (1–5 scale for speed, abundance) with new named sandbox options:

```lua
-- Old (commented out in current code):
-- if SandboxVars.Farming == 1 then ...

-- New:
local sandboxTime = getSandboxOptions():getOptionByName("FarmingSpeedNew"):getValue()
local sandboxYield = getSandboxOptions():getOptionByName("FarmingAmountNew"):getValue()
```

The mod does not patch the grow functions, only the props table, so this does not break the mod's Lua files. However, any existing saves that carried `SandboxVars.Farming` settings from earlier builds will not map to the new sandbox options — farming speed and yield in those saves will use defaults until reconfigured.

---

## 4. Crop Config Now Split Across Multiple Files (Load Order Risk)

Vanilla now loads crop props from two separate files instead of one:
- `farming_vegetableconf_vegetables.lua`
- `farming_vegetableconf_herbs.lua`

Both files write directly to `farming_vegetableconf.props`, the same table the mod patches. The mod's mechanism (direct table assignment) still works in principle, but **load order is critical**: if either vanilla file loads after the mod's files, vanilla values will overwrite the mod's overrides for any crops that appear in both.

The `42/` subfolder in the mod structure is the correct place for build-42-specific overrides, but whether the game's mod loader guarantees the mod loads last (after all base game scripts) should be verified. The herbs file (`farming_vegetableconf_herbs.lua`) defines new crops the mod doesn't touch, so that file poses no conflict — only `farming_vegetableconf_vegetables.lua` overlaps with the mod.

---

## 5. New Herb Crops Not Covered by the Mod

The game added 23 new farmable crops in `farming_vegetableconf_herbs.lua` (Basil, Chamomile, Leek, Rosemary, WildGarlic, etc.). The mod does not define or override any of these. They will use vanilla values, which is acceptable — but if the mod's design intent is to cover all farmable crops, these are gaps.

New crops also introduce two new properties the mod has never used:
- `isFlower = true` — flags a crop as a flower type
- `deerBane = true` — a pest repellent property not present on any mod crop
