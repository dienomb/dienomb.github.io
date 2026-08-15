# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A GitHub Pages site (`dienomb.github.io`) serving a personal blog. There is **no build system, no package manager, no test suite, and no dependencies** — it is static HTML + one CSS file, served directly from `main`.

## The critical constraint: `blog/` and `assets/` are generated output

Every file under `blog/` and `assets/` is machine-generated and pushed wholesale by an external publisher **that does not live in this repo**. Those commits appear as `chore: publish blog site` authored by `github-actions[bot]`, and each one rewrites the entire site (see `git show --stat` on any publish commit).

Consequences:

- **Hand-edits to `blog/**/index.html` or `assets/blog.css` are destroyed on the next publish.** If asked to change post content, styling, or page structure, say so — the fix belongs in the upstream generator, not here. Only make edits here if the user explicitly wants a throwaway/local change.
- Do not "tidy" the generated HTML. The stray blank lines, odd indentation, and HTML-entity-encoded non-ASCII (`&#233;` for `é`, `&#128564;` for the emoji) are template-engine artifacts, not defects.
- Git history for `blog/`/`assets/` is not a useful record of intent — it is a series of full-tree republishes interleaved with full-tree deletions.

Human-maintained files: `.github/workflows/blog-clean.yml`, `README.md`, `LICENSE`, this file.

## Workflow: `blog-clean.yml`

Manual-dispatch only (`workflow_dispatch`). It `rm -rf blog assets`, commits `chore: clean blog and assets folders`, and pushes to `main` with a 3-attempt fetch/rebase/push retry loop. It exists so the publisher can start from an empty tree — the clean/publish pairs in the log are that cycle. Checkout uses `persist-credentials: false` and the push re-points `origin` with `${{ github.token }}`; keep it that way.

## Site structure and conventions

- No `index.html` at the repo root — the live entry point is `/blog/`.
- One directory per post: `blog/<slug>/index.html`, with `blog/index.html` as the listing page (one featured post + a `More posts` grid).
- Images: `assets/blog/<slug>/hero-1.img` and `inline-N.img`. The `.img` extension is used regardless of the real format (the bytes are JPEG/PNG). Post pages reference them with `../../assets/...`, the index with `../assets/...`.
- Some posts ship with no assets directory at all, so their `<img>` tags 404 — e.g. `blog/dormir-bien-despues-de-los-30/` references three images that do not exist. This is an upstream publisher bug; several commits in history are failed attempts to patch it by hand.

## The CSS design system (`assets/blog.css`)

A single ~1100-line stylesheet drives every page. Pages select their appearance through data attributes set on both `<html>` and `<body>`:

- `data-theme` — `modern` is the base (defined on `:root`); `tech`, `editorial`, `bold`, and `minimal` are override blocks lower in the file. All current pages use `modern`.
- `data-topic` — `general` (default), plus `tech`, `culture`, `food`, `sports` accent overrides.
- `data-layout` + BEM-ish modifier classes on elements: `post-shell--{narrow,medium,wide,essay-quote,immersive-feature}`, `post-hero--{standard-hero,split-hero,full-bleed,image-led}`, `post-card--*`, `featured-post--*`. All current pages use `editorial-classic`.
- `data-image-placement` on `<body>` of post pages selects the hero treatment.

Design tokens live in `:root` (fonts `--font-display`/`--font-body`/`--font-mono`, palette `--ink*`/`--paper*`/`--surface`/`--rule*`, `--accent*`, `--shadow-*`, `--r-*` radii). Theme blocks work by redefining those tokens plus targeted component overrides — follow that pattern rather than hardcoding colors.

Fonts come from a Google Fonts `@import` at the top of the file, so rendering differs offline.

## Previewing

Open `blog/index.html` directly in a browser, or serve the repo root and hit `/blog/`:

```bash
python -m http.server 8000   # then http://localhost:8000/blog/
```

Relative asset paths resolve correctly under both.
