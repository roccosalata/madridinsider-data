# Leads Tracker — Madrid Insider Partnership Pipeline

> **`data/leads.json`** is the discovery queue for Madrid Insider.
> It tracks English-language Madrid resources that have been identified
> as potential partners or content sources, **before** they are promoted
> to full records in `records.json`.

## Purpose

The leads file solves three problems:

1. **Discovery is the bottleneck, not writing.** A 2026-07-22 mining pass
   found that 33 of 35 verified-active English-language Madrid resources
   (94%) were missing from the site. The leads file captures these
   finds in a structured queue.
2. **Partnership pipeline.** Each lead is classified by partnership tier
   and outreach status, so we can systematically work through them
   instead of forgetting about promising contacts.
3. **Source of truth for mining.** The schema is designed so future
   mining passes (manual or automated) can append new leads without
   duplicating existing ones.

## Schema

Each lead object in `data/leads.json`:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `id` | string | yes | `lead-{slug}`. Unique. |
| `name` | string | yes | Display name. |
| `url` | string | yes | Official URL. |
| `category` | string | yes | One of: `relocation`, `clubs`, `tours`, `cultural`, `study-abroad`, `tefl`, `universities-english`, `facebook-groups`, `business-services`, `legal`, `healthcare`, `family`, `other`. |
| `english_level` | string | yes | Same 4-tier system as records: `english-primary`, `english-available`, `english-partial`, `spanish-only`. |
| `source` | string | yes | How discovered: `web_search`, `user_tip`, `existing_record_audit`, `partner_referral`, `wayback_history`. |
| `discovered` | date | yes | ISO 8601 date. |
| `verified_active` | boolean | yes | Confirmed live within last 30 days. |
| `partnership_tier` | enum | yes | See tiers below. |
| `contact_email` | string or null | yes | Contact email if known, else `null`. |
| `outreach_status` | enum | yes | One of: `not_contacted`, `contacted`, `responded`, `promoted_to_record`, `declined`, `unresponsive`. |
| `notes` | string | yes | Free-form notes about the lead. |

## Partnership Tiers

In order of escalation (each tier builds on the previous):

| Tier | What it is | Effort | Revenue potential |
| --- | --- | --- | --- |
| `not_partnered` | Just a directory listing — no engagement. | None | None |
| `link_exchange` | We link to them, they link to us. | Low | SEO only |
| `cross_promo` | Co-post on social, newsletter mention, joint event. | Medium | Audience growth |
| `featured_listing` | Paid placement at top of subcategory. | Low | €50–200/mo |
| `referral_candidate` | Commission on conversions (bookings, enrollments, signings). | Medium | €100–500/conversion |
| `co_branded_content` | Joint guide, joint event, joint webinar. | High | Brand + revenue share |

Tier assignment is a *target*, not a current state — many leads start
at `referral_candidate` and the outreach determines what materializes.

## Mining Process

### Source categories (high-yield, ranked)

Based on the 2026-07-22 mining pass, these categories reliably yield
English-language Madrid resources:

1. **Relocation / buyer's agent services** — High partnership potential (referral commissions on €2-5K fees).
2. **TEFL / CELTA certification schools** — High (student referrals on €1,200-1,800 enrollments).
3. **Study-abroad program providers** — High (US university pipelines, seasonal cohorts).
4. **Facebook / Reddit expat communities** — High (audience reach, content amplification).
5. **English-language tour operators** — Medium (affiliate booking commissions).
6. **Expat social clubs & associations** — Medium (cross-event promotion, member discounts).
7. **Universities with English-taught degrees** — Medium (international student recruitment).
8. **English-language cultural centers** — Low-Medium (content cross-promotion).

### Categories to mine next (not yet covered)

- English-speaking lawyers / notaries / tax advisors
- English-language therapists / counselors
- International chambers of commerce (AmCham Spain, British Chamber)
- English-speaking gyms / fitness studios
- English-language churches (Anglican, international)
- English-speaking pet services / veterinarians
- English-language summer camps for kids
- International schools (already covered in `bilingual-schools-madrid`)
- English-friendly banks / financial advisors (partial coverage in `banking`)

### Search patterns that work

For each source category, the following search queries have proven
high-yield (run via `z-ai function -n web_search`):

```
"Madrid relocation services English speaking expats real estate"
"American Womens Club Madrid British Society International Newcomers Club"
"English language tour companies Madrid Devour Madrid Food Tour Sandemans"
"British Council Madrid Instituto Franklin English cultural center programs"
"Madrid expats Facebook groups Americans Brits international community size"
"TEFL certification Madrid Spain programs 2026 active"
"English language university programs Madrid bilingual degree Spain"
```

### Promotion criteria (lead → record)

A lead is promoted to a full record in `records.json` when **all** of
the following are true:

1. The lead has been **verified active** within the last 30 days.
2. The lead fits **one of the 5 categories** and one of the existing
   subcategories (no new subcategory needed — or a new subcategory has
   been added to `categories.json` first).
3. The lead adds **informational value** beyond what existing records
   cover (no duplication).
4. We can write at least **3 structured sections** for the record
   (text/callout/stats/checklist/list/steps/do_dont/table).

When a lead is promoted:

1. Create the full record in `records.json`.
2. Update the lead's `outreach_status` to `promoted_to_record`.
3. Add a `notes` field on the lead referencing the new record ID.
4. Update `related_records` on any existing records that should
   cross-link to the new one.

## Validation

The `scripts/sync-data.mjs` validator in the production repo currently
does **not** validate `leads.json`. This is intentional — leads are a
working file, not a published artifact, and we want to be able to add
raw finds quickly without strict validation overhead. If a lead is
promoted to a record, the standard record validator catches any schema
issues at that point.

## File shape

```json
{
  "version": 1,
  "updated": "2026-07-22",
  "leads": [
    { "id": "lead-...", ... },
    ...
  ]
}
```

The `version` field allows future schema migrations. The `updated`
field is the date of the most recent lead addition or status change.
