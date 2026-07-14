# Findings

## Mod Architecture
- KeysInLoot modifies SPT loot distributions during postDBLoad phase.
- Dynamically fetches keys and keycards from database using `ItemHelper`.
- Keys are sorted by their existing `RarityPvE` property defined on item templates.
- Supports both global weights and per-map rarity-based configurations.

## Config Overrides
- Modifies `itemcountDistribution` for Jackets, Duffle Bags, and Dead Scavs.
- Increases Jacket `cellsH` and `cellsV` to 3x3 to fit increased loot.
- Blanket nerfs key and keycard Flea/Trader sell prices.

## Vanilla Loot Mechanics & Math
- **Total Pool:** A vanilla Jacket (in SPT 3.11 Customs) has a total `relativeProbability` pool of roughly **200,053**.
- **Junk Weights:** Common items like matches and bolts range from 4,000 to 6,500 weight.
- **Key Weights:** 
  - Very Common (Naliv tech key): ~14,000
  - Common (OLI Storeroom): ~2,300
  - Uncommon (Danexert key): ~550
  - Rare (Weapon Testing key): ~300
- **Mod Impact:** Setting `keyWeight` to 500 assigns all 200+ missing keys an "Uncommon" rarity baseline. This inflates the total pool by 100,000 (making it 300,053) and results in roughly a 33% chance per slot to roll a key. Because all missing keys receive the same global 500 weight, their distinct rarities are flattened.
