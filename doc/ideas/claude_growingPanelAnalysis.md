# Growing Panel (Agriculture Tab) — Root Cause Analysis

**Created:** 2026-06-13  
**Version:** 42.19.0  
**Symptom:** With `PseudoAgriculture` mod enabled, the Literature UI "Agriculture" tab shows only wild garlic. With the mod disabled, all vegetable season recipes appear normally.

---

## What the Panel Actually Does

The Agriculture tab lives in `media/lua/client/ISUI/ISLiteratureUI.lua`. There are two relevant functions:

### `ISLiteratureUI:setLists()` — runs when the panel opens (line 419–429)

```lua
for typeOfSeed,props in pairs(farming_vegetableconf.props) do
    table.insert(typeOfSeedList, { typeOfSeed = typeOfSeed, ... })
end
-- adds every entry in farming_vegetableconf.props to the list
```

This populates the list with **every key currently in `farming_vegetableconf.props`** at the moment the panel opens.

### `ISLiteratureGrowingList:doDrawItem()` — runs for each row (lines 208–251)

```lua
local prop = farming_vegetableconf.props[item.text]
if not prop then return y end
if not prop.seasonRecipe then return y end
if not self.character:isRecipeActuallyKnown(prop.seasonRecipe) then return y end
-- then draws the row; errors here cause height=0 (silent skip)
local text = getText("Farming_" .. item.text)
text = text .. "<LINE>" .. ISFarmingMenu.plantInfo(prop)
```

A row is **only drawn** if all three conditions pass:
1. The crop key exists in `farming_vegetableconf.props`
2. `prop.seasonRecipe` is non-nil
3. `character:isRecipeActuallyKnown(prop.seasonRecipe)` returns true

If `ISFarmingMenu.plantInfo(prop)` throws a Lua error (e.g., nil concatenation), the row is also silently skipped.

### `ISFarmingMenu.plantInfo(prop)` — called from draw (lines 467–512)

```lua
text = text .. getText("Farming_Tooltip_MinWater") .. " " .. prop.waterLvl
text = text .. " <LINE>" .. getText("Farming_Tooltip_TimeOfGrow") ..
       math.floor((prop.timeToGrow * prop.harvestLevel) / 24 * calcNextTimeFactor())
```

This requires both `prop.waterLvl` and `prop.harvestLevel` to be non-nil. If either is nil, Lua throws "attempt to perform arithmetic on a nil value" and the crop is silently skipped.

---

## What `isRecipeActuallyKnown` Actually Checks

`IsoGameCharacter.java:10181–10183`:
```java
public boolean isRecipeActuallyKnown(String string) {
    return this.isRecipeKnown(string, true);
}
```

`IsoGameCharacter.java:10166–10175`:
```java
public boolean isRecipeKnown(String string, boolean bl) {
    Recipe recipe = ScriptManager.instance.getRecipe(string);
    if (recipe == null) {
        // Season recipes are NOT in ScriptManager as old-style Recipe objects
        // bl=true means SeeNotLearntRecipe bypass does NOT apply here
        return this.getKnownRecipes().contains(string);
    }
    return this.isRecipeKnown(recipe);
}
```

Season recipes (`"base:broccoli growing season"` etc.) are **not registered as old-style `Recipe` objects** in ScriptManager — they are CraftRecipes or unregistered strings. So `getRecipe()` returns null, and the check falls back to `character.getKnownRecipes().contains(string)`.

**The panel shows a crop only if `"base:xxx growing season"` is literally in the character's known recipes list.**

---

## Why Wild Garlic Shows and Vegetables Don't

Wild garlic is defined in `farming_vegetableconf_herbs.lua`. The mod does **not** touch herbs. Its seasonRecipe is `"base:wild garlic growing season"`.

All 32 vegetables are defined in `farming_vegetableconf_vegetables.lua` and **overridden by the mod**.

The "only wild garlic" observation means one of:

### Theory A: The character has not actually learned the vegetable season recipes

The character knows `"base:wild garlic growing season"` but not `"base:broccoli growing season"` etc. This would happen if:
- The character was created/tested without reading vegetable seed packets in the current session
- "Reading recipes" in the UI's Recipes tab is different from **learning** them (the Recipes tab with MetaKnowledge shows all recipes from items regardless of whether the character has read the actual items)

The Recipes tab (listbox3) pulls from `ScriptManager.getAllItems()` and shows all literature items' `LearnedRecipes` without requiring the character to have actually read the item. The Agriculture tab requires actual character knowledge.

### Theory B: The mod changes `prop.seasonRecipe` to a value that doesn't match what the character learned

Ruled out: both game-folder mod files (`C:\Users\edwar\Zomboid\mods\PseudoAgriculture\`) have correct `"base:xxx growing season"` format for all 32 crops (verified by grep: 32 seasonRecipe entries in each file).

### Theory C: `farming_vegetableconf.props` lacks vegetable entries when the panel opens

If the Lua state is re-initialized during world loading and mod files are not re-executed, vegetable props would revert to vanilla values. But vanilla has the same `seasonRecipe` format, so this would still work for a character who knows vanilla-format recipes.

This cannot explain the "only wild garlic" symptom unless there's also a character knowledge gap.

### Theory D: `ISFarmingMenu.plantInfo(prop)` is silently failing for vegetables

`plantInfo` requires `prop.waterLvl` and `prop.harvestLevel`. Both are present in all 32 mod crop entries (verified: 32 `harvestLevel` and 68 `waterLvl` matches across 32 crops, ~2 per crop due to commented-out old values). Ruled out.

---

## Most Likely Root Cause

**Theory A is most likely.** The Agriculture tab requires the character to have literally read a seed packet item in-game, not just viewed the Recipes tab. Wild garlic may have been learned from a previous session or from finding wild garlic seeds in the world.

The symptom "I read recipes for a bunch of the vegetables" may mean the user viewed them in the **Recipes tab** (listbox3, which shows all literature-item recipes regardless of character knowledge), not that they physically used a seed packet item in inventory.

However, this does not fully explain why the behavior differs between mod-enabled and mod-disabled — if the character has no vegetable season knowledge in both cases, neither should show vegetables.

---

## Key Open Question

**Does reading a seed packet item (double-clicking it or reading it from inventory) actually add the season recipe to the character's knownRecipes when the mod is enabled?**

The `ISInventoryPaneContextMenu.lua:1254` check uses `doesSeasonRecipeExist(v)` to decide whether to show "Show Growing Seasons" as a context option. If `doesSeasonRecipeExist` returns false (because props is somehow empty at that moment), this option doesn't appear — but this is only a context menu option, not the learn mechanism itself.

The actual learn mechanic is Java-side: when the character "reads" an item with `LearnedRecipes`, the Java engine calls `character.learnRecipe(string)` directly. This does **not** call `doesSeasonRecipeExist`. So the mod should not block recipe learning through this path.

---

## Recommended Diagnostic Test

To confirm or rule out Theory A, try this in-game with the mod enabled:

1. Open the Literature UI and go to the Agriculture tab — note what shows
2. Find a Broccoli seed packet item in inventory, right-click → Read (or double-click to use it)
3. Open the Literature UI again — does Broccoli now appear in the Agriculture tab?

If yes: the character simply hadn't learned the recipes yet. The mod is working correctly. The "only wild garlic" state occurs because wild garlic was previously learned but vegetable seeds haven't been read yet in this session.

If no: the learn mechanism is broken when the mod is enabled. The next step would be adding console debug output to `learnRecipe` or checking whether the seed packet read action is actually firing the learn event.

---

## Status of DevCycle 001 Phases

- **Phase 1 (seasonRecipe format):** Complete. Both game-folder mod files confirmed correct.
- **Phase 2 (load order):** Still unverified. If Theory A is confirmed by the test above, Phase 2 becomes the remaining risk to check.
- **Phase 3 (badMonthHardyLevel):** Still planning.

---

## Files Examined

| File | Lines | Purpose |
|------|-------|---------|
| `media/lua/client/ISUI/ISLiteratureUI.lua` | 208–251, 339–429 | Agriculture tab rendering and population |
| `media/lua/client/Farming/ISUI/ISFarmingMenu.lua` | 467–512 | `plantInfo(prop)` — requires waterLvl, harvestLevel |
| `zombie/characters/IsoGameCharacter.java` | 10166–10183 | `isRecipeActuallyKnown` — falls back to knownRecipes.contains() |
| `C:\Users\edwar\Zomboid\mods\PseudoAgriculture\42\media\lua\server\RealisticKentuckyFarming.lua` | all | 32 crops, all have correct seasonRecipe, waterLvl, harvestLevel |
| `C:\Users\edwar\Zomboid\mods\PseudoAgriculture\common\media\lua\server\PseudoGPTFarming.lua` | all | 32 crops, same fields confirmed |
