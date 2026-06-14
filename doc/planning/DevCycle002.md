# DevCycle 002: Almanac-Based timeToGrow Corrections

**Status:** Work Complete  
**Start Date:** 2026-06-14  
**Target Completion:** TBD  
**Focus:** Update `timeToGrow` values for 7 crops to match real-world grow times sourced from almanac.com

---

## Goal

Several crops in the mod have `timeToGrow` values that don't match real-world growing times. Using almanac.com as a reference, 7 crops have been identified with inaccurate values. This cycle corrects those values in both mod Lua files to improve agricultural realism.

## Desired Outcome

The 7 affected crops have updated `timeToGrow` values consistent with almanac.com data. Both `RealisticKentuckyFarming.lua` and `PseudoGPTFarming.lua` are updated in sync. The `doc/crop_calendar.md` DaysToGrow column reflects the new values.

---

## Tasks

### Phase 1: Update timeToGrow in RealisticKentuckyFarming.lua

**Status:** Work Complete

- [x] Update Broccoli: 432 → 288
- [x] Update Carrots: 360 → 336
- [x] Update Cauliflower: 432 → 360
- [x] Update Kale: 336 → 288
- [x] Update Greenpeas: 288 → 264
- [x] Update Spinach: 192 → 144
- [x] Update Turnip: 264 → 240

**Technical Notes:**  
File: `PseudoAgriculture/42/media/lua/server/RealisticKentuckyFarming.lua`  
The `timeToGrow` value in game units converts to in-game days by dividing by 4.8.

| Crop         | Old timeToGrow | New timeToGrow | Old Days | New Days | Almanac Days |
|--------------|---------------|----------------|----------|----------|--------------|
| Broccoli     | 432           | 288            | 90       | 60       | 60           |
| Carrots      | 360           | 336            | 75       | 70       | 70           |
| Cauliflower  | 432           | 360            | 90       | 75       | 75           |
| Kale         | 336           | 288            | 70       | 60       | 60           |
| Green Peas   | 288           | 264            | 60       | 55       | 55           |
| Spinach      | 192           | 144            | 40       | 30       | 30           |
| Turnip       | 264           | 240            | 55       | 50       | 50           |

### Phase 2: Update timeToGrow in PseudoGPTFarming.lua

**Status:** Work Complete

- [x] Update Broccoli: 2160 → 288
- [x] Update Carrots: 1800 → 336
- [x] Update Cauliflower: 2160 → 360
- [x] Update Kale: 1680 → 288
- [x] Update Greenpeas: 1440 → 264
- [x] Update Spinach: 960 → 144
- [x] Update Turnip: 1320 → 240

**Technical Notes:**  
File: `PseudoAgriculture/common/media/lua/server/PseudoGPTFarming.lua`  
Per DC001 findings, both files define full crop configurations independently. Any value change must be applied to both files or the common version will silently override the 42-specific one depending on load order.

### Phase 3: Update crop_calendar.md

**Status:** Work Complete

- [x] Update DaysToGrow column for all 7 affected crops in `doc/crop_calendar.md`

**Technical Notes:**  
The DaysToGrow values in the calendar doc are derived from `timeToGrow / 4.8`. After Phase 1 and 2 are complete, the column values for the 7 crops must be updated to match.

### Phase 4: Sync all remaining timeToGrow values in PseudoGPTFarming.lua

**Status:** Work Complete

- [x] Barley: 1440 → 972
- [x] Bell Pepper: 1680 → 336
- [x] Cabbages: 2160 → 432
- [x] Corn: 2160 → 432
- [x] Cucumber: 1200 → 240
- [x] Flax: 2400 → 480
- [x] Garlic: 5040 → 1008
- [x] Hemp: 2160 → 432
- [x] Hops: 2880 → 576
- [x] Lettuce: 1440 → 288
- [x] Onion: 2160 → 432
- [x] Potatoes: 1920 → 384
- [x] Pumpkin: 2640 → 528
- [x] Radishes: 600 → 120
- [x] Rye: 2880 → 972
- [x] Soybeans: 1920 → 384
- [x] Strawberryplant: 1200 → 240
- [x] SugarBeets: 2160 → 432
- [x] Sunflower: 1920 → 384
- [x] SweetPotato: 2400 → 480
- [x] Tobacco: 2880 → 576
- [x] Tomato: 1440 → 288
- [x] Watermelon: 2160 → 432
- [x] Wheat: 2880 → 972
- [x] Zucchini: 1200 → 240

**Technical Notes:**  
File: `PseudoAgriculture/common/media/lua/server/PseudoGPTFarming.lua`  
Target values are taken from `RealisticKentuckyFarming.lua`, which is the authoritative source for this mod's crop timings. The 7 crops updated in Phases 1–2 are already in sync and excluded here. This phase completes full parity between the two files for `timeToGrow`.

---

## Notes and Risks

- Source: almanac.com (accessed 2026-06-14). Values represent typical days to maturity under good growing conditions.
- Both Lua files must be kept in sync — see DC001 notes on load order risk.
- No season data (`sowMonth`, `badMonth`, `bestMonth`, `riskMonth`) is being changed in this cycle.

---

## Completion Summary

*Fill in when the cycle closes. Move this document to `doc/planning/completed/` afterward.*

**Completion Date:** 2026-06-14  
**Phases Completed:** All  
**Work Deferred:** None  

**Accomplishments:**  
- Updated `timeToGrow` for 7 crops in `RealisticKentuckyFarming.lua` to match almanac.com real-world grow times
- Synced the same 7 almanac values into `PseudoGPTFarming.lua`
- Updated `doc/crop_calendar.md` DaysToGrow column for the 7 affected crops
- Synced all remaining 25 crops in `PseudoGPTFarming.lua` to match `RealisticKentuckyFarming.lua`

**Metrics:**  
- Files modified: 3
- Crops corrected (almanac): 7
- Crops synced between files (Phase 4): 25

**Lessons / Notes:**  
PseudoGPTFarming.lua had significantly inflated timeToGrow values across all 32 crops — roughly 3–5x higher than RealisticKentuckyFarming.lua. The two files are now fully in sync. Future timeToGrow changes should be made in RealisticKentuckyFarming.lua first, then mirrored to PseudoGPTFarming.lua.

