# Item Insertion — What We Need To Know (Zero Guesswork)

## Current Status
Binary insertion works — offsets, TOC, trailing sizes, timestamps verified.
Template system working — 575 templates from real saves.
Live game memory dump working — 5,900 items with stackLimit from version.dll.

## The Chart — Every Field We Write vs What The Game Expects

| # | Field | What We Set | What Game Expects | Status | Source |
|---|-------|------------|-------------------|--------|--------|
| 1 | `type_index` | Schema lookup at runtime | Per-save schema index for ItemSaveData | ✅ SOLVED | save_parser.py schema |
| 2 | `mask` (3 bytes) | From template DB | Must match item type (varies per item) | ✅ SOLVED (template) | Template DB has exact mask per itemKey |
| 3 | `_saveVersion` | 1 | Always 1 | ✅ SOLVED | Confirmed in all saves |
| 4 | `_itemNo` | max + 100 | Globally unique u64 across all item lists | ✅ SOLVED | Code scans all items for max |
| 5 | `_itemKey` | User picks | Must exist in game (5,900 valid keys) | ✅ SOLVED | Live dump has all 5,900 keys |
| 6 | `_slotNo` | 0 | Vendor: 0, Inventory: next free | ✅ SOLVED | Handled in code |
| 7 | `_stackCount` | User picks | Must be ≤ stackLimit | ✅ SOLVED | **item_limits.json: 5,900 items with stackLimit from live game memory** |
| 8 | `_endurance` | From template | 0xFFFF for all sold items | ✅ SOLVED (template) | Verified: all 167 sold items = 0xFFFF |
| 9 | `_sharpness` | From template | Only present if mask includes bit 8 | ✅ SOLVED (template) | Template has correct mask → correct fields |
| 10 | `_maxSocketCount` | From template | 5 for all items in saves | ⚠️ PARTIAL | All items in saves have 5. DLL reads 1 (wrong offset). Using template value. |
| 11 | `socket_type_index` | Schema lookup at runtime | Per-save index for ItemSocketSaveData | ✅ SOLVED | save_parser.py schema |
| 12 | `_transferredItemKey` | = `_itemKey` (parse tree offset) | EQUALS _itemKey | ✅ SOLVED | Verified all 167 sold items. Parse tree gives exact offset. |
| 13 | `_chargedUseableCount` | From template | Non-zero (15 for most, varies) | ✅ SOLVED (template) | Template preserves real game value |
| 14 | `_timeWhenPushItem` | From template | Game timestamp | ✅ SOLVED (template) | Template preserves real game timestamp |
| 15 | `_timestamp` (parallel) | 0 | Parallel to _storeSoldItemDataList | ✅ SOLVED | Code inserts matching timestamp entry |
| 16 | `01 01` bytes at +201/202 | From template | Socket list footer bytes | ✅ SOLVED | Not a field — part of socket list encoding |

## Data Sources Available

| Source | Items | Fields | Status |
|--------|-------|--------|--------|
| Template DB (from saves) | 575 | Full binary (mask, all fields, sockets) | ✅ Working |
| item_limits.json (from DLL) | 5,900 | stackLimit, typeFlags | ✅ Working |
| item_names.json (from web) | 6,059 | name, category, internalName | ✅ Working |
| iteminfo.pabgb (from PAZ) | 5,993 | name, stackLimit, equipment class | ✅ Parsed |
| Live game memory (DLL) | 5,900 | stackLimit, typeFlags (more with better offsets) | ✅ Working |

## What's Still Needed

### For the DLL (version.dll deep probe)
- [ ] Fix `equipClassHash` offset — currently reading 0 for all items. Need to walk Pointer A correctly.
- [ ] Fix `maxSocketCount` offset — currently reading 1 for all. ptrB+0x04 is wrong.
- [ ] Fix `slotType` — all 0xFFFF. Offset +0x44 may have shifted.
- [ ] Fix `inventoryCategory` — only shows 1 and 2. Offset +0xA4 is wrong.
- [ ] Add raw memory dump command for specific items to probe the struct layout

### For item insertion to be fully reliable
- [ ] Validate stackCount ≤ stackLimit at insertion time (data is available)
- [ ] When no template exists, need to determine correct mask from iteminfo data
- [ ] Community template collection to reach 90%+ coverage (currently 9.6%)

## Stack Limit Data (from live game memory — VERIFIED)

| Stack Limit | Item Count | Examples |
|-------------|-----------|---------|
| 0 | 15 | Virtual currency (camp resources) |
| 1 | 4,420 | Equipment, weapons, armor, tools |
| 5 | 616 | Crafting materials |
| 10 | 289 | Food, consumables |
| 20 | 319 | Potions, materials |
| 50 | 202 | Trade goods, materials |
| 99 | 1 | Special item |
| 100 | 35 | Ammo (arrows) |
| 999 | 2 | Special stackable |
| 100,000 | 1 | Silver (money) |
