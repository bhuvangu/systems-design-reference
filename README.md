# Systems Design Reference

A set of dense, print-ready reference sheets for system design and software architecture. View them in the browser or download the PDFs.

**Live site:** `https://<your-username>.github.io/systems-design-reference/`
*(replace `<your-username>` after you publish — see below)*

---

## What's inside

| # | Sheet | Format | What it covers |
|---|-------|--------|----------------|
| 01 | **Architecture Cookbook Cheatsheet** | 2 pages | 25 patterns with variants + tradeoffs |
| 02 | **Systems Engineering Keyword Map** | 2 pages | ~400 concept terms in 10 areas |
| 03 | **The Architect's Vocabulary** | 2 pages | ~110 terms with one-line definitions |
| 04 | **Practitioner's Field Notes** | 4 pages | 18 areas: term · gloss · ⚠ gotcha |

Plus two long-form companions in [`docs/`](docs/): the full prose **Architecture Cookbook** and a 54-chapter **System Design concept TOC**.

---

## Publish to GitHub Pages (one time, ~2 minutes)

From inside this folder:

```bash
git init
git add .
git commit -m "Systems design reference sheets"
git branch -M main
# create an empty repo named systems-design-reference on github.com first, then:
git remote add origin https://github.com/<your-username>/systems-design-reference.git
git push -u origin main
```

Then turn on Pages:

1. Go to your repo on github.com → **Settings** → **Pages**
2. Under **Build and deployment → Source**, choose **Deploy from a branch**
3. Branch: **main**, folder: **/ (root)** → **Save**
4. Wait ~1 minute, then visit `https://<your-username>.github.io/systems-design-reference/`

That's it — `index.html` is the landing page and everything links from there.

> **Note:** the `.nojekyll` file tells GitHub Pages to serve the files as-is (no Jekyll build), which is what we want for a plain static site.

---

## Repository layout

```
.
├── index.html              ← landing page (links everything)
├── sheets/                 ← the cheat sheets (open in browser)
│   ├── cookbook-cheatsheet.html
│   ├── keyword-map.html
│   ├── vocabulary.html
│   └── field-notes.html
├── pdf/                    ← pre-rendered print-ready PDFs
│   ├── cookbook-cheatsheet.pdf
│   ├── keyword-map.pdf
│   ├── vocabulary.pdf
│   └── field-notes.pdf
├── docs/                   ← long-form markdown companions
│   ├── architecture-cookbook.md
│   └── system-design-toc.md
├── .nojekyll
├── LICENSE
└── README.md
```

## Editing a sheet

Each sheet is a single self-contained `.html` file (inline CSS, web fonts from Google Fonts). Edit the HTML directly, then to regenerate a PDF use your browser's **Print → Save as PDF** with “Background graphics” on, sized to A4.

## License

[MIT](LICENSE) — use, adapt, and share freely.
