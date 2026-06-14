# Crop Days to Grow Analysis
**Version**: Project Zomboid 42.11.0  
**Created**: 2026-06-14  
**Source**: `media/lua/server/Farming/farming_vegetableconf_vegetables.lua`

## Formula Discovery

The in-game "Days to Grow" is not stored directly. It is derived from three fields:

```
DaysToGrow = floor(timeToGrow × harvestLevel / 24)
```

- `timeToGrow` — game-hours per growth stage
- `harvestLevel` — the growth stage at which the crop is harvested (5 = mature, 6 = fullGrown)
- `24` — game-hours per in-game day
- Result is floored (truncated), not rounded

`mature` and `fullGrown` are always 5 and 6 respectively for all vegetables. The key variable that changes total grow time is `harvestLevel`: crops with `harvestLevel = 6` must complete one extra stage before harvest.

## Full Crop Table

| Crop | `timeToGrow` | `harvestLevel` | Exact Days | Floored Days |
|------|-------------|---------------|------------|-------------|
| Radishes | 144 | 5 | 30.00 | **30** |
| Broccoli | 292 | 5 | 60.83 | **60** |
| Bell Pepper | 292 | 5 | 60.83 | **60** |
| Cabbage | 292 | 5 | 60.83 | **60** |
| Cauliflower | 292 | 5 | 60.83 | **60** |
| Green Peas | 292 | 5 | 60.83 | **60** |
| Kale | 292 | 5 | 60.83 | **60** |
| Lettuce | 292 | 5 | 60.83 | **60** |
| Spinach | 292 | 5 | 60.83 | **60** |
| Sugar Beets | 292 | 5 | 60.83 | **60** |
| Turnip | 292 | 5 | 60.83 | **60** |
| Strawberry | 360 | 6 | 90.00 | **90** |
| Tomato | 360 | 6 | 90.00 | **90** |
| Carrots | 432 | 5 | 90.00 | **90** |
| Corn | 432 | 5 | 90.00 | **90** |
| Cucumber | 432 | 5 | 90.00 | **90** |
| Flax | 432 | 6 | 108.00 | **108** |
| Onion | 432 | 5 | 90.00 | **90** |
| Potatoes | 432 | 5 | 90.00 | **90** |
| Pumpkin | 432 | 5 | 90.00 | **90** |
| Rye | 432 | 6 | 108.00 | **108** |
| Soybeans | 432 | 5 | 90.00 | **90** |
| Sunflower | 432 | 5 | 90.00 | **90** |
| Sweet Potato | 432 | 5 | 90.00 | **90** |
| Barley | 432 | 6 | 108.00 | **108** |
| Watermelon | 432 | 5 | 90.00 | **90** |
| Wheat | 432 | 6 | 108.00 | **108** |
| Zucchini | 432 | 5 | 90.00 | **90** |
| Tobacco | 866 | 5 | 180.42 | **180** |
| Hemp | 960 | 6 | 240.00 | **240** |
| Hops | 960 | 6 | 240.00 | **240** |
| Garlic | 1152 | 5 | 240.00 | **240** |

*Leek not included — defined in `farming_vegetableconf_herbs.lua`, not yet analyzed.*

## Key Findings

### The harvestLevel Effect
Crops with `harvestLevel = 6` (Barley, Flax, Rye, Wheat, Hemp, Hops, Tomato, Strawberry) must grow through all 6 stages before harvest. Crops with `harvestLevel = 5` harvest at the mature stage. This is the sole reason Barley (432, harvestLevel=6) takes **108 days** while Carrots (432, harvestLevel=5) take **90 days** despite having identical `timeToGrow` values.

### The timeToGrow = 292 Anomaly
All 11 crops with `timeToGrow = 292` produce a non-integer result (60.83 days). A value of `288` would yield a clean 60 days. The extra 0.83 day per crop is discarded by floor truncation. This appears to be an intentional or incidental quirk in the vanilla values.

### Crops That Share the Same Days to Grow
- **30 days**: Radishes only
- **60 days**: Broccoli, Bell Pepper, Cabbage, Cauliflower, Green Peas, Kale, Lettuce, Spinach, Sugar Beets, Turnip
- **90 days**: Strawberry, Tomato, Carrots, Corn, Cucumber, Onion, Potatoes, Pumpkin, Soybeans, Sunflower, Sweet Potato, Watermelon, Zucchini
- **108 days**: Barley, Flax, Rye, Wheat
- **180 days**: Tobacco
- **240 days**: Hemp, Hops, Garlic
