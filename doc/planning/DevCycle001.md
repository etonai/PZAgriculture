# DevCycle 001: Restore Compatibility with 42.19.0

**Status:** COMPLETE
**Start Date:** 2026-06-13  
**Target Completion:** TBD  
**Focus:** Fix all structural breakages introduced by game changes in build 42.19.0

---

## Goal

The mod stopped working when Project Zomboid updated to 42.19.0. Two structural changes in the game's farming system broke the mod: the `seasonRecipe` ID format was changed engine-wide, and crop configuration was split across new files with an unknown load order relative to mods. This cycle fixes those issues so the mod functions correctly again.

## Desired Outcome

All crops defined in `PseudoGPTFarming.lua` and `RealisticKentuckyFarming.lua` correctly link to their season recipes in-game. The load order risk is understood and mitigated. The mod runs without errors on 42.19.0.

---

## Tasks

### Phase 1: Fix `seasonRecipe` IDs

**Status:** Work Complete

Update every `seasonRecipe` string in both mod files from the old Title Case format to the new namespaced lowercase format the engine now expects.

- [x] Update all `seasonRecipe` values in `PseudoAgriculture/common/media/lua/server/PseudoGPTFarming.lua`
- [x] Update all `seasonRecipe` values in `PseudoAgriculture/42/media/lua/server/RealisticKentuckyFarming.lua`

**Technical Notes:**  
The engine changed `seasonRecipe` IDs from `"Crop Growing Season"` to `"base:crop growing season"` across all vanilla crops. The full mapping is documented in `doc/ideas/AgricultureChanges_42_19_0.md`. Both mod files must be updated — `PseudoGPTFarming.lua` covers the common version and `RealisticKentuckyFarming.lua` covers the 42-specific overrides, and both contain full crop definitions with the old format.

The `minVegAutorized` / `maxVegAutorized` property names in the mod files are intentionally spelled without the 'h'. That matches the engine's own spelling (`props.minVegAutorized`, `props.maxVegAutorized`) as read in `SFarmingSystem.lua`. Do not change these.

### Phase 2: Verify Load Order

**Status:** UNNECESSARY - Bad idea by Claude

Confirm that the game loads base game crop scripts before mod scripts, so the mod's `farming_vegetableconf.props` table assignments overwrite vanilla values rather than the reverse.

- [ ] Test in-game that a modded crop's `sowMonth` reflects mod values, not vanilla values
- [ ] Document the confirmed load order behavior

**Technical Notes:**  
The game split crop config across `farming_vegetableconf_vegetables.lua` and `farming_vegetableconf_herbs.lua` in 42.19.0. Both write to `farming_vegetableconf.props`, the same table the mod patches. If either file loads after the mod, vanilla values silently overwrite the mod. The `42/` subfolder is the expected place for build-42 overrides and should load after base scripts, but this needs in-game verification. The herbs file poses no conflict since the mod defines no herbs.

### Phase 3: Evaluate `badMonthHardyLevel`

**Status:** DECIDED AGAINST

Determine whether the mod needs to set `badMonthHardyLevel` on Carrots and Kale.

- [ ] Test in-game behavior of Carrots and Kale without `badMonthHardyLevel` set
- [X] Decide whether to add the property or leave it absent
- [ ] If adding, determine appropriate skill level values consistent with the mod's design intent

**Technical Notes:**  
Vanilla 42.19.0 added `badMonthHardyLevel` to two crops: Carrots (level 4) and Kale (level 3). This gates planting in bad months behind a farming skill requirement. The mod overrides both crops but sets no value for this property. Engine behavior when the property is absent is unknown — it may default to 0 (no gate) or produce unexpected results. This phase is low priority but should be resolved before marking the cycle Verified.

---

## Open Questions

1. **Does the engine silently ignore a missing `badMonthHardyLevel` or does it cause an error?**  
   Recommendation: Test in-game with Carrots or Kale during a bad month before deciding whether to add the property.

---

## Notes and Risks

- The `seasonRecipe` fix in Phase 1 is the only confirmed hard breakage. Phases 2 and 3 are verification and low-risk edge cases.
- Both mod files (`PseudoGPTFarming.lua` and `RealisticKentuckyFarming.lua`) define the full set of crop overrides independently. Any fix must be applied to both files.
- The 23 new herb crops added in 42.19.0 are not covered by this mod. They will use vanilla values. Covering them is out of scope for this cycle.

---

## Completion Summary

*Fill in when the cycle closes. Move this document to `doc/planning/completed/` afterward.*

**Completion Date:** TBD  
**Phases Completed:**  
**Work Deferred:**  

**Accomplishments:**  

**Metrics:**  
- Files modified:  

**Lessons / Notes:**  
