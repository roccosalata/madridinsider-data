# Data Schema

This document defines the shape of every JSON file in `data/`. The production
site ([`roccosalata/madridinsider`](https://github.com/roccosalata/madridinsider))
mirrors these files at build time via the `npm run sync-data` script.

All record IDs are unique across the whole `data/` directory and lowercase
with hyphens (kebab-case). Adding a new field is fine; removing one requires
a code change in the consumer.

---

## `data/categories.json`

The 5 permanent homepage categories, each with a list of subcategories.
URL slugs are derived from `subcategory.slug` (falls back to `subcategory.id`).

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `id` | string | yes | Lowercase slug, unique. One of `essentials`, `living`, `see`, `do`, `now`. |
| `title` | string | yes | Display name. |
| `description` | string | yes | One-sentence subtext. |
| `emoji` | string | yes | A single emoji. |
| `color` | enum | yes | One of `red`, `blue`, `green`, `purple`, `orange`. |
| `subcategories` | array | yes | 4–8 subcategory objects (see below). |

Each subcategory object:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `id` | string | yes | Lowercase slug, unique within the category. |
| `slug` | string | recommended | URL segment (e.g. `transportation` for `/essentials/transportation`). Falls back to `id` if missing. |
| `title` | string | yes | Display name. |
| `icon` | string | yes | A single emoji used as the icon. |
| `summary` | string | yes | One-sentence summary. |

URL pattern: `/{category.id}/{subcategory.slug}` (or `/{category.id}/{subcategory.id}` if no slug).

---

## `data/records.json`

The canonical record database. Every record represents one searchable, linkable
thing about Madrid — a metro line, a museum, a procedure, a venue.

### Required fields

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `id` | string | yes | Lowercase, kebab-case. Unique across the whole file. Used as URL slug. |
| `category` | enum | yes | One of the 5 category IDs: `essentials`, `living`, `see`, `do`, `now`. |
| `subcategory` | enum | yes | Must match a subcategory ID inside the chosen category. |
| `title` | string | yes | Human-readable name. |
| `summary` | string | yes | 1–2 sentence elevator pitch. Shown in lists and search results. |
| `content` | string | yes | Full body. 1–3 paragraphs. |
| `last_updated` | date | yes | ISO 8601 date, `YYYY-MM-DD`. |
| `status` | enum | yes | One of `evergreen`, `annual review`, `dynamic`, `legacy`, `archived`. |
| `official_url` | string | yes | An official, primary URL (museum, transit, embassy, government, or city portal). Syntactically valid `https://` URL. |
| `english_friendly` | boolean | yes | `true` if the resource has any English availability (tiers 1-3). `false` for tier 4 (Spanish-only). Must be consistent with `english_level` if both are set. |
| `english_level` | enum | recommended (optional but encouraged) | One of `english-primary`, `english-available`, `english-partial`, `spanish-only`. See English availability tiers below. Defaults to `english-available` (if `english_friendly=true`) or `spanish-only` (if `false`) when missing. |
| `related_records` | array | yes | List of record IDs (kebab-case). All referenced IDs must exist. May be empty. |

### Optional fields

| Field | Type | Notes |
| --- | --- | --- |
| `tags` | string[] | Free-form labels for search and filtering. Lowercase, kebab-case. |
| `sections` | Section[] | Structured content blocks. When present, the record page renders these instead of the flat `content` string. `content` is still required (it powers search and is the fallback). See below. |

### Structured `sections` (added 2026-07-20)

Each section is an object with a `type` discriminator and type-specific fields.

| `type` | Required fields | Optional | Use for |
| --- | --- | --- | --- |
| `text` | `heading`, `body` | — | Plain prose (intro, history, explanation) |
| `callout` | `heading`, `body` | `tone` (`info` \| `warning` \| `success` \| `danger`, default `info`) | Highlighted tip, warning, danger |
| `stats` | `heading`, `items[]` | — | Quick facts (cost, hours, validity) |
| `checklist` | `heading`, `items[]` | — | Required documents, prep list |
| `list` | `heading`, `items[]` | — | Tips, highlights, named items |
| `steps` | `heading`, `items[]` | — | Step-by-step process |
| `do_dont` | `heading`, `do[]`, `dont[]` | — | Etiquette, rules |
| `table` | `heading`, `columns[]`, `rows[][]` | — | Prices, hours, comparisons |

Item shapes:

- `stats.items[]`: `{ "label": string, "value": string }`
- `checklist.items[]` and `list.items[]`: `{ "title": string, "body"?: string, "meta"?: string[] }`
- `steps.items[]`: `{ "title": string, "body": string, "tip"?: string }`
- `do_dont.do[]` and `dont[]`: `string[]`
- `table.columns[]`: `string[]`, `table.rows[][]`: `string[][]`

Example:

```json
"sections": [
  { "type": "callout", "heading": "Important", "body": "You need an appointment.", "tone": "warning" },
  { "type": "stats", "heading": "Quick Facts", "items": [
    { "label": "Cost", "value": "€9.64" },
    { "label": "Processing", "value": "Same day" }
  ]},
  { "type": "steps", "heading": "Process", "items": [
    { "title": "Book appointment", "body": "Visit the gov website.", "tip": "Check at 9 AM daily." }
  ]}
]
```

The validator in `scripts/sync-data.mjs` (production repo) enforces all of this at build time.

### Status values

| Status | When to use |
| --- | --- |
| `evergreen` | Doesn't change much (museums, neighbourhoods, traditions). |
| `annual review` | Updates yearly (festivals, opening hours, prices). |
| `dynamic` | Changes frequently (transit alerts, current exhibitions, ETIAS status). |
| `legacy` | Historical content kept for reference. |
| `archived` | No longer accurate. Hidden from the main UI. |

### English availability tiers (added 2026-07-20)

Every record should be classified into one of 4 tiers via the optional
`english_level` field. This implements the project's English-first content
priority — English-friendly records surface above Spanish-only ones in
search results and record listings.

| Tier | `english_level` value | What it means | Badge |
| --- | --- | --- | --- |
| 1 | `english-primary` | Official source is primarily in English, OR has a complete English version with full content parity (US Embassy, Prado /en) | 🌟 EN★ (gold) |
| 2 | `english-available` | Official source has a dedicated English version (may be incomplete). Default for `english_friendly=true` records without an explicit tier. | 🟢 EN (green) |
| 3 | `english-partial` | Some English info available, most content Spanish. | 🟡 EN~ (amber) |
| 4 | `spanish-only` | No English from the official source. We describe it in English on Madrid Insider but the user needs Spanish/Translate. `english_friendly` must be `false`. | ⚪ ES (grey) |

**Consistency rule**: if both `english_level` and `english_friendly` are set,
they must agree — tiers 1-3 require `english_friendly=true`, tier 4 requires
`english_friendly=false`. The build validator enforces this.

### URL pattern

`/{category}/{subcategory.slug-or-id}/{record.id}`

Examples:

- `/essentials/transportation/metro-de-madrid`
- `/living/healthcare/english-speaking-doctors`
- `/see/museums/museo-del-prado`
- `/do/food/mercado-san-miguel`
- `/now/events/veranos-de-la-villa`

### The 25 Golden Records

The first 25 records to be authored are listed in
[`GOLDEN_RECORDS.md`](https://github.com/roccosalata/madridinsider/blob/main/GOLDEN_RECORDS.md)
in the production repo. They cover the most-asked-about entities and are
treated as required entries in `records.json` for a successful build.

---

## `data/events.json` (legacy — being phased out)

The "what's on this week" event cards. As of 2026 the site prefers rendering
events from the `events` subcategory of `now` in `records.json` (which can
include dates, venues, and status). Keep this file only for backward
compatibility; new events should go into `records.json`.

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `id` | string | yes | Unique, prefix `evt-`. |
| `title` | string | yes | Event name. |
| `date` | string | yes | Human-friendly, e.g. `15 Jul`. |
| `day` | string | yes | Numeric day. |
| `month` | string | yes | 3-letter month, uppercase. |
| `venue` | string | yes | Where it happens. |
| `category` | enum | yes | `Music`, `Culture`, `Food`, `Sport`, or `Family`. |
| `free` | boolean | yes | `true` if free entry. |

---

## `data/quickAccess.json` (legacy)

10 quick-access shortcuts. Will be migrated to records shortly.

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `id` | string | yes | Unique, prefix `qa-`. |
| `title` | string | yes | Card heading. |
| `description` | string | yes | Card subtext. |
| `emoji` | string | yes | A single emoji. |
| `href` | string | yes | In-page anchor or URL. |

---

## Validation

A record is invalid if any of the following is true:

- Missing any required field.
- `id` is not unique.
- `category` is not one of the 5 approved values.
- `(category, subcategory)` does not match a defined subcategory.
- `status` is not one of the 5 approved values.
- `related_records` contains an ID that doesn't exist.
- `official_url` is missing or not a syntactically valid URL.
- `last_updated` is not ISO 8601 (`YYYY-MM-DD`).

Run validation locally with:

```bash
python3 -c "import json; r=json.load(open('data/records.json')); print(len(r), 'records')"
```

The production CI also runs a strict JSON parse on every push.

---

## Versioning

This repo is a content-only data store. The data file format is implicitly
versioned by the consumer. The production site pins to the schema documented
above; when that schema changes, coordinate the update in both repos
(`madridinsider-data` and the mirror in `madridinsider/data/`).
