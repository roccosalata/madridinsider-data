# Data Schema

This document defines the shape of every JSON file in `data/`. The production
site ([`roccosalata/madridinsider`](https://github.com/roccosalata/madridinsider))
reads these files (or a mirror of them) at build time.

All files are **JSON arrays of objects**. IDs are strings and must be unique
within their file. Adding a new field is fine; removing one requires a code
change in the consumer.

---

## `data/categories.json`

The 5 main homepage category cards.

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `id` | string | yes | Lowercase slug, unique. Used for in-page anchors. |
| `title` | string | yes | Card heading. |
| `description` | string | yes | Card subtext, 1-2 sentences. |
| `emoji` | string | yes | A single emoji character. |
| `color` | enum | yes | One of `red`, `blue`, `green`, `purple`, `orange`. |
| `bullets` | string[] | yes | 3-5 short phrases. |

Example:

```json
{
  "id": "essentials",
  "title": "Madrid Essentials",
  "description": "Everything you need to know before you arrive and during your stay.",
  "emoji": "🎒",
  "color": "red",
  "bullets": ["Visa & entry requirements", "Currency, tipping & budgets"]
}
```

---

## `data/events.json`

"What's on this week" — the event cards on the homepage.

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `id` | string | yes | Unique, prefix `evt-`. |
| `title` | string | yes | Event name. |
| `date` | string | yes | Human-friendly, e.g. `15 Jul`. |
| `day` | string | yes | Numeric day, 1-2 chars, no leading zero. |
| `month` | string | yes | 3-letter month, uppercase (`JUL`, `AUG`). |
| `venue` | string | yes | Where it happens. |
| `category` | enum | yes | `Music`, `Culture`, `Food`, `Sport`, or `Family`. |
| `free` | boolean | yes | `true` if free entry. |

---

## `data/quickAccess.json`

The 10 "Quick access" cards on the homepage.

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `id` | string | yes | Unique, prefix `qa-`. |
| `title` | string | yes | Card heading. |
| `description` | string | yes | Card subtext, one short sentence. |
| `emoji` | string | yes | A single emoji character. |
| `href` | string | yes | In-page anchor (`#essentials`) or absolute URL. |

---

## Versioning

This repo is a content-only data store. The data file format is implicitly
versioned by the `version` field at the top of each file (when present) and
by which consumer is reading it. The production site pins to a specific
shape — when that shape changes, coordinate the update in both repos.
