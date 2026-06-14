# Investigation Handoff: PseudoAgriculture Mod — Missing Farming Recipes

**Created:** 2026-06-13  
**Game Version:** 42.19.0  
**Mod ID:** `RealisticKentuckyFarmingCalendar`  
**Mod folder (game-loaded):** `C:\Users\edwar\Zomboid\mods\PseudoAgriculture\`  
**Mod folder (dev):** `C:\dev\zomboid\zomboid_42_x\mymods\PseudoAgriculture\`  
**Note:** These two locations are NOT symlinked — they are independent copies. Any fix must be applied to both.

---

## The Problem

With the mod enabled, the Literature UI's Agriculture tab shows only **wild garlic**. With the mod disabled, all expected vegetable season recipes appear normally.

The user also confirmed: console log (recorded during a mod-enabled session) shows all 32 vegetable season recipes failing verification at world load.

---

## Investigation Summary

### What Was Wrong Previously

A prior conversation diagnosed the cause as "old Title Case format" for `seasonRecipe` strings (e.g., `"Broccoli Growing Season"` instead of `"base:broccoli growing season"`). This was called Phase 1 and is documented in `doc/planning/DevCycle001.md`. Phase 1 has been applied to both mod locations. The format is now correct in all files. **Phase 1 did not resolve the visible symptom.**

A second claim from that conversation was that the mod was loading from a wrong folder path (`mods\RealisticKentuckyFarmingCalendar\`). This was also wrong — the game loads from `mods\PseudoAgriculture\`, and that folder has the Phase 1 fix applied.

### Confirmed Facts

1. **Both game-loaded mod files have correct `seasonRecipe` format.**  
   Verified by grep on `C:\Users\edwar\Zomboid\mods\PseudoAgriculture\`:
   - `common/media/lua/server/PseudoGPTFarming.lua` — 32 entries, all `"base:xxx growing season"`
   - `42/media/lua/server/RealisticKentuckyFarming.lua` — 32 entries, same format

2. **Both mod files have `waterLvl` and `harvestLevel` for all 32 crops.**  
   32 `harvestLevel` matches and 68 `waterLvl` matches (≈2 per crop, one being a commented-out old value) in each file.

3. **The Agriculture tab rendering code is in `ISLiteratureUI.lua`.**  
   - `setLists()` (line 419) iterates `farming_vegetableconf.props` and adds every key to the list  
   - `doDrawItem()` (line 208) has three sequential gates before drawing a row:
     ```lua
     if not prop then return y end
     if not prop.seasonRecipe then return y end
     if not self.character:isRecipeActuallyKnown(prop.seasonRecipe) then return y end
     ```
   - After those gates, calls `ISFarmingMenu.plantInfo(prop)` (line 227) which needs `prop.waterLvl` and `prop.harvestLevel` — both confirmed present in all mod entries

4. **`isRecipeActuallyKnown` falls back to a string-contains check.**  
   `IsoGameCharacter.java:10166–10175`: season recipes are not registered as old-style `Recipe` objects in ScriptManager, so `getRecipe()` returns null and the method falls back to `character.getKnownRecipes().contains("base:broccoli growing season")`. A row only appears if the literal string is in the character's known recipes.

5. **Console verification errors are debug-only and do not disable recipes.**  
   `ScriptManager.java`: `VerifyAllCraftRecipesAreLearnable()` is guarded by `if (!Core.bDebug) { return; }`. The 32 failures logged at console lines 495–526 are diagnostic only. The line 527 message — *"Craft Recipe Learning possible issues detected; this may be a false positive"* — is the game itself acknowledging this may not be a real problem.

6. **Wild garlic is a herb, not a vegetable.**  
   Defined in `farming_vegetableconf_herbs.lua:563`. The mod touches only vegetables. Wild garlic appearing while vegetables don't means the character knows `"base:wild garlic growing season"` but not any vegetable season recipe — at the time the panel is opened.

### What Was Ruled Out

- **Deployment problem** (wrong folder): Ruled out. Correct files are in the game-loaded path.
- **seasonRecipe format mismatch**: Ruled out. All 32 crops in both files have correct format.
- **Missing `waterLvl` or `harvestLevel`**: Ruled out. All 32 crops have both fields.
- **Key name mismatch** (case differences): Ruled out. Mod uses same PascalCase keys as vanilla (e.g., `Broccoli`, `Barley`).
- **Java verification errors blocking recipes**: Ruled out. That code path is debug-only and only logs.

### What Remains Unresolved

**The core question: why does the character know `"base:wild garlic growing season"` but not any vegetable season recipes when the mod is enabled?**

Two competing explanations:

**Explanation A — Character knowledge gap (not a mod bug):**  
The character hasn't physically read vegetable seed packet items in the current session. Wild garlic was learned previously (prior session, or from finding wild garlic seeds). The Literature UI's Recipes tab shows all seed-packet recipes whether or not the character has read the item — but the Agriculture tab requires the character to have actually used the item. The user may have been viewing the Recipes tab and assuming those represented learned knowledge.

**Explanation B — Lua load order overwriting mod props:**  
If vanilla files (`farming_vegetableconf_vegetables.lua` or `farming_vegetableconf_herbs.lua`) load AFTER the mod files, vanilla values overwrite the mod's values. But since vanilla also has correct `seasonRecipe` format, this would not affect which recipes the character knows — it would only affect seasonal window values (sowMonth, etc.), not recipe visibility.

**Explanation C — `farming_vegetableconf.props` is partially cleared during world loading:**  
The herbs file starts with `require "Farming/farming_vegetableconf"`. If PZ's Lua `require` re-executes files (non-standard behavior), this would reset `props = {}` at world load time after vegetables were populated but before herbs were re-added. However, this would affect vanilla equally — it cannot explain why the behavior differs between mod-enabled and mod-disabled.

---

## Pending Diagnostic Test

Before continuing code analysis, the following in-game test should be run with the mod enabled and confirmed:

1. Start a new game (or use a character with no farming recipe knowledge)
2. Open the Literature UI → Agriculture tab → note that only wild garlic shows
3. Find a **Broccoli seed packet** in inventory → right-click → Read (or double-click to use)
4. Reopen the Literature UI → Agriculture tab

**Result (confirmed 2026-06-13):** Broccoli did **not** appear after reading the seed packet with the mod enabled. Explanation A (character knowledge gap) is ruled out. **This is a real bug.** The learn mechanism is broken or the Agriculture tab check is failing for a reason not yet identified.

The next investigation step is to determine whether `character.learnRecipe()` is being called with the correct string, or whether some check between "right-click seed packet" and `learnRecipe()` is gated on `doesSeasonRecipeExist()` or another props-dependent function.

---

## DevCycle 001 Phase Status

| Phase | Description | Status |
|-------|-------------|--------|
| Phase 1 | Fix `seasonRecipe` format to `"base:xxx growing season"` | **Complete** |
| Phase 2 | Verify load order — confirm modded `sowMonth` values are in effect | **Planning** |
| Phase 3 | Evaluate `badMonthHardyLevel` for Carrots and Kale | **Planning** |

Phase 2 is independent of the diagnostic test above. Even if the Agriculture tab issue resolves as a knowledge gap, Phase 2 still needs in-game verification that, e.g., Barley shows `sowMonth = {8, 9, 10}` (Kentucky values) not `{8, 9, 10}` vanilla or `{2, 3, 4, 5, 6, 7, 8, 9, 10}` (PseudoGPT values). The Kentucky file overrides the common file (42/ loads after common/), so Kentucky values should win — but this needs confirmation in game.

---

## Key File Reference

| File | Why It Matters |
|------|---------------|
| `media/lua/client/ISUI/ISLiteratureUI.lua` | Agriculture tab rendering — `setLists()` and `doDrawItem()` |
| `media/lua/client/Farming/ISUI/ISFarmingMenu.lua:467` | `plantInfo(prop)` — needs `waterLvl` and `harvestLevel` |
| `zombie/characters/IsoGameCharacter.java:10166` | `isRecipeActuallyKnown` — falls back to knownRecipes.contains() |
| `media/lua/server/Farming/farming_vegetableconf.lua:392` | `farming_vegetableconf.props = {}` — resets props table |
| `media/lua/server/Farming/farming_vegetableconf_vegetables.lua` | Vanilla vegetable props — no `require` at top |
| `media/lua/server/Farming/farming_vegetableconf_herbs.lua:1` | `require "Farming/farming_vegetableconf"` at top — potential reset trigger |
| `zombie/scripting/ScriptManager.java:831` | `VerifyAllCraftRecipesAreLearnable` — debug-only, does not disable recipes |
| `docs/console.txt:495–527` | 32 vegetable season recipe failures during world load |
| `doc/planning/DevCycle001.md` | Work tracking for this fix cycle |
| `claudeDocs/claude_growingPanelAnalysis.md` | Detailed code analysis of the Agriculture tab rendering pipeline |
