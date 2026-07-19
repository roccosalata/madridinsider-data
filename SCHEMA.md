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
| `english_friendly` | boolean | yes | `true` if the resource explicitly serves English speakers. |
| `related_records` | array | yes | List of record IDs (kebab-case). All referenced IDs must exist. May be empty. |

### Optional fields

| Field | Type | Notes |
| --- | --- | --- |
| `tags` | string[] | Free-form labels for search and filtering. Lowercase, kebab-case. |

### Status values

| Status | When to use |
| --- | --- |
| `evergreen` | Doesn't change much (museums, neighbourhoods, traditions). |
| `annual review` | Updates yearly (festivals, opening hours, prices). |
| `dynamic` | Changes frequently (transit alerts, current exhibitions, ETIAS status). |
| `legacy` | Historical content kept for reference. |
| `archived` | No longer accurate. Hidden from the main UI. |

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
