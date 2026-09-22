# Torah Worldwide — progress map (מפת ההתקדמות)

The achievements board of https://www.torah-worldwide.com. A static page, no build step,
served by GitHub Pages from the `main` branch, repository root.

| file | what it is | who edits it |
|---|---|---|
| `data.json` | **the content** — every tile: the root, the branches with their task nodes, and Tomer's verbatim wording per item | anyone updating progress (Hermes) |
| `index.html` | the design — CSS, the inline sprite images, layout and interaction code. It fetches `data.json` at load and computes everything else (counters, percentage, tier tallies, wires, the AI brief) | only for design changes |
| `.nojekyll` | tells GitHub Pages to serve the files as-is | never |

## data.json

```json
{
  "root":     { "id": "root", "name": "…", "date": "YYYY-MM-DD", "tier": "platinum", "art": "root", "tex": "home", "desc": "…" },
  "branches": [
    { "label": "הטקסט", "row": -4, "from": "root", "tex": "klaf", "nodes": [ …node, node, node… ] },
    …
  ],
  "verbatim": { "1": "Tomer's exact words for item 1", "2": "…", … }
}
```

`branches` is **the task list**. Each branch is a chain of tiles hanging off the node named in
`from` (`"root"` or any node `id`); `row` is its vertical band on the board; node order inside
a branch is the real chronological order and decides the tile's position in the tree.

### Node fields

| field | required | meaning |
|---|---|---|
| `id` | yes | unique string. Open items use `"L<n>"`. |
| `lock` | open items | the **item number**. Present ⇒ the tile is locked (grey frame, lock badge, number tab, dashed wire). |
| `was` | closed items | the item number after it shipped — replaces `lock`. Keeps the number traceable and keeps the verbatim text on the card. |
| `date` | every unlocked node | `"YYYY-MM-DD"` — the day Tomer approved it. |
| `name` | yes | short Hebrew title, shown under the tile. |
| `desc` | yes | one or two plain Hebrew sentences — the hover tooltip and the card. |
| `tier` | yes | `"platinum"` \| `"gold"` \| `"silver"` \| `"bronze"`. |
| `art` | no | key of the `SPRITES` map in `index.html` (a small picture in the tile). |
| `glyph` | no | one Hebrew letter, shown when there is no sprite for `art`. A new item with no artwork needs only `glyph`. |
| `tex` | no | tile texture; defaults to the branch's `tex`. |
| `status` | no | array of Hebrew strings for an open item that was worked on and **not** finished. Shown on the card under „מה נעשה ומה נשאר", adds the „חלקי" tag, and goes into the AI brief as PARTIAL. |
| `updates` | no | array of Hebrew strings, each starting with its date — later fixes or follow-ups on a tile that already shipped (Tomer's rule, 22.9: a fix to an existing thing is a line on its tile, not a new item). Shown on the card under „המשך הדרך", no tag; listed in the AI brief as LATER FIXES. |

`verbatim` is keyed by item number as a string (`"6"`). The card shows it under „במילים שלך" for
every node whose `lock` or `was` matches. **Never paraphrase, tidy or shorten these — they are
Tomer's own words.**

## Recipes

**Close an item (it shipped and Tomer approved it):** in its node change `"lock": 6` to
`"was": 6` and add `"date": "2026-09-22"`. Leave everything else — id, name, tier, art/glyph,
desc, position — untouched. Rewrite `desc` only if what shipped genuinely differs from the plan.

```json
{ "id": "L6", "was": 6, "date": "2026-09-22", "name": "מילה קריאה על הקלף", "tier": "silver", "art": "l6", "glyph": "מ", "desc": "…" }
```

**Add an item:** append a node with the next unused number at the **end** of the right branch's
`nodes`, and add its wording to `verbatim` under the same number.

```json
{ "id": "L16", "lock": 16, "name": "…", "tier": "silver", "glyph": "…", "desc": "…" }
```

**Record a later fix on a shipped tile:** append a dated Hebrew line to the node's `updates` array
(create it if missing). Do not change `date` or `desc`, do not add a new item.

**Mark an open item as partially done:** keep `lock`, add a `status` array (see the field table;
item 8 in the data is the worked example).

## Rules that must not drift

- **Item numbers are permanent.** Never renumber, never reuse, never reorder existing nodes.
- **The map is not a to-do list.** A locked tile is a trophy not yet earned; it unlocks only on Tomer's approval.
- `data.json` must stay strict JSON — no comments, no trailing commas. Check before committing:
  `python -m json.tool data.json > /dev/null` (or `node -e "JSON.parse(require('fs').readFileSync('data.json','utf8'))"`).
- Do not edit `index.html` to change content.

## Notes

- The page requests `data.json` with `cache: "no-cache"`, so an edit on `main` is visible as soon as
  GitHub Pages has rebuilt (usually well under a minute).
- Local preview needs a web server, because `fetch` does not work from `file://`:
  `python -m http.server 8000` in this folder, then open http://localhost:8000/.
- Source history: this page was the Claude artifact "מפת ההתקדמות של תורה וורלדוויד" (built from
  `torah-v2-roadmap/board_template.html` on Tomer's PC). From now on this repository is the live copy.
