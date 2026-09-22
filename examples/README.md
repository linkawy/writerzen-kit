# Real output

Three trimmed runs, so you can see the shape of the data before spending a credit on your own. All
three came from live calls on 2026-09-22 for `ecommerce platform` in the United States, in English.

| File | Command | What to look at |
|---|---|---|
| `keyword-explorer.json` | `wz kw "ecommerce platform"` | `search_volume`, `competition`. `average_cpc` is **not** dollars here |
| `topic-discovery.json` | `wz topic "ecommerce platform"` | `headlines` and `related` inside each entry's `value` |
| `keyword-planner.json` | `wz plan create --file …` | `clusters[].items[]`, and `urls` — the results the grouping is computed from |

Each file is cut down: 40 of 1,518 keyword ideas, 3 of 100 topic entries, and one whole six-keyword
plan. Nothing else is edited.

Two things these files show that the documentation only asserts:

- `average_cpc` for `ecommerce platform` is `280058` in the Explorer file and `12.18` in the Planner
  file. Same keyword, same country, same day — the two tools do not report CPC in the same unit.
- In `topic-discovery.json`, compare an entry's `topic` and `relevant` against its `headlines`. The
  headlines are specific and useful; the label and the score sitting on top of them are the parts
  SKILL.md tells an agent to throw away.
