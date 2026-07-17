# Madrid Insider — Data

> 🗄️ **The content "database" for the live Madrid Insider site.**
> Categories, events, and quick-access entries, stored as plain JSON for
> easy editing — no CMS, no admin UI, just a PR.

The live site at [madridinsider.com](https://www.madridinsider.com/) reads
this content (or a mirror of it) at build time. To change what users see
on the live site — add an event, update a category, fix a typo — you
edit a file here and open a PR. Mavis or a maintainer will sync it into
the production repo.

---

## 🔗 The four-repo family

| Repo | Role |
| --- | --- |
| [roccosalata/madridinsider](https://github.com/roccosalata/madridinsider) | **Production live site** — multi-collaborator, what users see |
| [roccosalata/madridinsider-explorer](https://github.com/roccosalata/madridinsider-explorer) | Mavis's dev sandbox for new code |
| **[roccosalata/madridinsider-data](https://github.com/roccosalata/madridinsider-data)** | **This repo** — content "database" (JSON) |
| [roccosalata/madridinsider-lab](https://github.com/roccosalata/madridinsider-lab) | Throwaway testbed for risky experiments |

### The flow

```
content edit (data)  ──►  review + merge  ──►  mirror into production repo  ──►  live site
```

1. **Edit a JSON file** in `data/`.
2. **Open a PR** — keep it focused (one event, one category, one bullet).
3. **Get a review** from at least one other collaborator.
4. **Merge** — the production repo's mirror of these files is updated, and
   the next deploy picks them up.

See [SCHEMA.md](./SCHEMA.md) for the exact shape of each file.

---

## 📁 Layout

```
madridinsider-data/
├── data/
│   ├── categories.json     # 5 homepage category cards
│   ├── events.json         # "What's on this week" event cards
│   └── quickAccess.json    # 10 "Quick access" cards
├── archive/
│   └── legacy-static-site.html   # previous static site, kept for reference
├── SCHEMA.md               # data shape documentation
└── README.md
```

---

## ✏️ Editing

Pick a file, follow [SCHEMA.md](./SCHEMA.md), and edit. Validation happens
in the production consumer, so stick to the existing keys and types unless
you also plan to update the consumer.

### Add a new event

1. Open `data/events.json`.
2. Add a new object to the array with a unique `id` (continue the `evt-N` numbering).
3. Pick the right `category` from the enum in SCHEMA.md.
4. Open a PR titled e.g. `content(events): add Real Madrid vs. Sevilla on 3 Aug`.

### Update a category

1. Open `data/categories.json`.
2. Edit the matching object's `title`, `description`, or `bullets`.
3. Open a PR.

### Add a quick-access card

1. Open `data/quickAccess.json`.
2. Add a new object with a unique `id` (`qa-N`).
3. Use an in-page anchor (`#essentials`, `#see`, etc.) or an absolute URL for `href`.
4. Open a PR.

---

## 🤝 Conventions

- **One logical change per PR** — don't bundle five event additions into one PR.
- **Keep keys sorted** the way the schema lists them.
- **Don't reformat the whole file** in the same PR that changes content — it
  makes review painful.
- **Use 2-space indent** and trailing newline.

---

## 🛡️ Safety

- This repo is public; assume anything you commit is visible.
- Don't put real personal info, unpublished embargoes, or paid-for content here.
- For typos: just fix them. For content disputes: open an issue first.

---

## 📄 License

MIT — fork, customize, ship.
