# Systems Design Reference

A static site with a single print-ready reference sheet plus two long-form markdown companions.

## Pages

- **Landing:** `https://<your-username>.github.io/<repo>/`
- **Field Notes (direct link):** `https://<your-username>.github.io/<repo>/docs/field-practitioner.html`

The field-notes page is intentionally **not linked** from the landing page — reach it by its direct URL.

## Field Notes

A single self-contained HTML file (inline CSS, web fonts only — no local dependencies). It's one continuous flowing document: the browser handles layout, and **Chrome's Print → Save as PDF** handles pagination (enable “Background graphics”). No fixed page sizing baked in.

18 areas of system-design terms in `term · gloss · ⚠ gotcha` form: clocks & ordering, cache & write strategies, locking & consistency, traffic shaping & routing, capacity & estimation, top-line metrics, scaling failure modes, deployment, messaging & queues, debugging & analytics, counting at scale, rate limiting, authentication, isolation, compliance, operations & incidents, tech finance & business.

## Companions (markdown)

- [`docs/architecture-cookbook.md`](docs/architecture-cookbook.md) — 25 patterns with problem, approaches, pros/cons, and how to choose.
- [`docs/system-design-toc.md`](docs/system-design-toc.md) — 54-chapter hierarchical map of system-design concepts.

## Layout

```
.
├── index.html                    ← landing page (no links out)
├── docs/
│   ├── field-practitioner.html   ← the reference sheet (single continuous file)
│   ├── architecture-cookbook.md
│   └── system-design-toc.md
├── .nojekyll
├── LICENSE
└── README.md
```

## License

[MIT](LICENSE)
