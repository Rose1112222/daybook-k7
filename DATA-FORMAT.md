# Tally data format

Everything is stored as named JSON documents. In the installed app they live in IndexedDB: database `tally`, object store `docs`, with the document name as the key. A backup file is one JSON object:

    { "app": "tally", "version": 1, "exported": "<ISO time>", "docs": { "<name>": { ... }, ... } }

Dates are always `YYYY-MM-DD` and mean the log day, which can differ from the calendar date late at night (see `roll` and `fin`). Weights of food are grams, or milliliters for foods measured by volume. Nutrient amounts use these short keys everywhere:

    k calories   p protein g   c carbohydrate g   f fat g   sf saturated fat g
    fi fiber g   su sugars g   na sodium mg       ch cholesterol mg

A missing nutrient key means zero or unknown.

## settings

| Field | Meaning |
|---|---|
| budget | Everyday calorie budget |
| meals | List of meal names; entries refer to a meal by its position in this list |
| unit | Body weight unit, `lb` or `kg` |
| maint | Maintenance calories (optional) |
| prof | Inputs remembered by the TDEE calculator: `sex` (`f`/`m`), `by` birth year, `h` height in cm, `act` activity multiplier |
| roll | Hour of the day (0 to 12) when a new log day starts. Default 6 |
| fin | The last log day the owner marked finished. While it is today or later, the current log day is the day after it |
| start | Date the "since start" weight comparison counts from |
| wy0 | Earliest year that has a weights document |
| lb | Calendar date of the last backup |
| remap5 | Legacy flag for a one-time meal renumbering; leave as is |

## foods

`{ "items": { "<id>": food } }`. A food:

| Field | Meaning |
|---|---|
| n, br | Name, brand |
| k p c f sf fi su na ch | Nutrients per 100 g (or per 100 ml when `un` is `ml`) |
| un | `ml` for foods measured by volume; absent means grams |
| sv, sn | Size of one serving in g or ml, and what a serving is called (absent means "Servings") |
| bc | List of barcode numbers, digits only with leading zeros removed |
| lg, lq, lu | Last amount logged: in base units, and as typed with its unit code |
| u | Time last used (ms), for ordering |
| src | `usda` or `usda-branded` when the food came from USDA data |
| nd | Nutrient keys the source had no data for |

Unit codes: `g`, `oz`, `lb`, `ml`, `floz`, `tbsp`, `cup`, and `sv` (servings).

## log-YYYY-MM

One document per month.

| Field | Meaning |
|---|---|
| v | Format marker, always 2 |
| days | `{ "<date>": [entry, ...] }` |
| x | `{ "<date>": calories }` exercise noted for the day (display only, never changes the budget) |
| b | `{ "<date>": calories }` a budget that applies to that one day |

An entry is a snapshot, so later edits to a food do not change the log:

| Field | Meaning |
|---|---|
| i | Entry id |
| m | Meal position in `settings.meals` |
| n | Food name at the time |
| g | Amount in grams or ml; `null` for a quick-add of calories only |
| un | `ml` when measured by volume |
| q, qu, qn | Amount as typed, its unit code, and the serving name, when it was not entered in the base unit |
| k p c f sf fi su na ch | Nutrient totals for this entry |
| fid, bid | Id of the food or batch it came from |
| d | 1 when eaten. Absent means planned. Planned entries still count toward the day |

## weights-YYYY

One document per year: `{ "w": { "<date>": weight } }` in the unit from settings.

## batches

`{ "items": { "<id>": batch }, "pots": [ { "n": name, "g": empty weight in grams } ] }`

A batch is a pot of cooked food:

| Field | Meaning |
|---|---|
| n, d | Name and date made |
| ing | Ingredient list; each item has the same shape as a log entry (without `m` and `d`) |
| w | Cooked weight in grams. Absent until it is entered; the batch cannot be logged from before then |
| k p c f sf fi su na ch | Per 100 g of the cooked food, worked out from the ingredients and `w` |
| lg, lq, lu | Last amount logged from it |
| done | Date it was finished. Finished batches only appear under "Make again"; one is kept per name |

## Outside the documents

`localStorage` key `tally-usda-key` holds the USDA API key. It is not part of backups.
