# Repository Guidelines

## Project Overview

`otix-site` is the marketing + blog website for **Otix.ai** (practical AI-assisted software delivery: spec-driven workflows, coding agents, the DONE framework). It is a **hand-authored static site** served by **GitHub Pages** on the custom domain **otix.ai** (`CNAME`). There is no application backend, no build step, and no generator. What you edit is what ships.

## Architecture & Data Flow

- Every page is a standalone `.html` file with an inline `<head>` (meta + Open Graph/Twitter + JSON-LD) and a **shared markup skeleton**: `<header class="site-header">` (masthead + `<nav class="site-nav">`) → `<main>` → `<footer>`. The header/footer/nav are copy-pasted verbatim into each page; there is no include/templating system, so a nav change means editing every page.
- Styling is two hand-written vanilla CSS files loaded in order: `styles.css` (design tokens + global/landing) then `blog.css` (blog listing + article typography). `blog.css` depends on tokens defined in `styles.css`, so keep that load order.
- "Data flow" is purely static: browser loads HTML → both stylesheets → Google Fonts (Inter) → `assets/logo.png`. SEO/agent crawlers read `robots.txt`, `sitemap.xml`, `llms.txt`, and the per-page JSON-LD.
- Relative paths matter: root pages link the blog as `blog/index.html`; pages under `blog/` reach the root as `../index.html` and load `../styles.css` / `../blog.css` / `../assets/logo.png`.

## Key Directories

- `/` (root): landing page `index.html`, global `styles.css`, SEO/deploy plumbing (`sitemap.xml`, `robots.txt`, `llms.txt`, `CNAME`).
- `blog/`: `index.html` (post listing), plus one `.html` file per post (`local-llm-agents.html`, `beyond-vibe-coding-hve.html`). `blog.css` lives at root but governs this directory.
- `assets/`: brand images (`logo.png`; `syle_guide.png` — note the misspelled filename, referenced nowhere in CSS).
- `.omc/`, `.omp/`: OMC/OMP agent tooling state — operational artifacts, not site content. Do not edit or document their internals.

## Development Commands

There is **no build, test, or lint tooling**. Workflow:

- **Preview locally:** `cd /Users/jrrall/Code/infrastructure/otix-site && python3 -m http.server 8000`, then open `http://localhost:8000/` and `http://localhost:8000/blog/`. (Use a server, not `file://`, so relative paths resolve.)
- **Deploy:** commit and `git push` to `main` (`git remote`: `https://github.com/otixai/otix-site.git`). GitHub Pages publishes `main` at `https://otix.ai`. No pipeline to run.

## Code Conventions & Common Patterns

- **Indentation:** 2 spaces, HTML5 `<!DOCTYPE html>`, `<html lang="en">`. Match the surrounding file; there is no formatter config.
- **CSS class naming:** flat, semantic, kebab-case (`site-header`, `blog-post`, `post-card`, `post-tags`, `tag`, `dek`, `subtitle`, `blog-back`). **No BEM, no utility classes.** Style nested elements with descendant selectors (`.post-card h2`, `.blog-post a`), not new classes.
- **State via attributes/pseudo-classes:** active nav uses `[aria-current="page"]` (not an `.active` class); interactivity via `:hover`/`:active`.
- **Design tokens (defined once in `styles.css :root`, reused everywhere):** `--rust-orange: #C8531D` (links/accents/hover — never substitute another color for links), `--rust-hover: #a8431a`, `--dark-navy: #2A2E36` (body text), `--haze-grey: #6B6B6B` (secondary/meta), `--pine-green: #476E55` (footer links), `--light-warm: #F8F6F3`, `--border-light: #E5E0DA`, `--iron-ridge: #3A3F3E`, `--off-white: #E8E2D9` (code text on dark `pre`). Reference as `var(--token)`; do not hardcode hex.
- **Type:** body `Inter` (Google Fonts weights 400/500/600/700/900, already preconnected — add no new font imports); code `'SF Mono', 'Fira Code', Menlo, monospace`. `h1`–`h3` sizes use `clamp()` for fluid scaling; do not hardcode px on headings. `.blog-post` body line-height is `1.8`.
- **Layout:** flexbox for header/nav/tags, CSS grid for `.pillars` (landing) and `.post-list` (blog). Text columns cap at `max-width: 680px`. Responsive breakpoints are max-width only: `640px`, `560px`, `380px`. Respect `prefers-reduced-motion`.
- **Writing voice (blog):** first-person, **Title Case `<h2>`/`<h3>` headings**, a `<p class="subtitle">` dek under the `<h1>`, and an italic closing note (`<p><em>…</em></p>`) before a `Resources` `<h3>`. Avoid AI-slop tells: em dashes (`&mdash;`), arrow glyphs (`&rarr;`), and empty intensifiers are actively removed from copy — prefer commas, colons, or separate sentences.
- **SEO plumbing (every page `<head>`):** `<link rel="canonical">`, full `og:*` + `twitter:*` tags, and a JSON-LD `@graph`. Index pages use `og:type=website`; blog posts use `og:type=article` plus `article:published_time`/`article:publisher`. Every page repeats the full `Organization` node with the shared `@id` `https://otix.ai/#organization`; posts add a `BlogPosting` (`@id` `<url>#article`) and a `BreadcrumbList`.

### Adding a new blog post (do all five, in order)

1. **Create `blog/<slug>.html`** by copying `blog/local-llm-agents.html`. Keep `<header>`/`<nav>`/`<footer>` verbatim. Update `<title>` (`"Title | Otix.ai"`), `<meta name="description">`, `canonical`, all `og:`/`twitter:` tags, `article:published_time`, and the JSON-LD `BlogPosting` fields (`headline`, `name`, `description`, `url`, `datePublished`, `dateModified`, `articleSection`, `keywords`).
2. **Write the body** inside `<article class="blog-post">`: `<a href="index.html" class="blog-back">&larr; All posts</a>`, then `<h1>`, `<p class="subtitle">`, `<div class="post-tags">` of `<span class="tag">`, `<hr>`, `<h2>`/`<h3>` sections, closing `<em>` note, `Resources`.
3. **Add a card** to the top of `<ul class="post-list">` in `blog/index.html` (reverse-chronological):
   ```html
   <li class="post-card">
     <h2><a href="<slug>.html">Post Title</a></h2>
     <p class="dek">One-line summary.</p>
     <div class="post-tags">
       <span class="tag">Tag One</span>
       <span class="tag">Tag Two</span>
     </div>
   </li>
   ```
4. **Add to the JSON-LD `blogPost` array** in `blog/index.html` (newest first): `{ "@type": "BlogPosting", "headline": "...", "url": "https://otix.ai/blog/<slug>.html", "datePublished": "YYYY-MM-DD" }`.
5. **Add a `<url>` to `sitemap.xml`** unless the post is `noindex` (see below):
   ```xml
   <url>
     <loc>https://otix.ai/blog/<slug>.html</loc>
     <lastmod>YYYY-MM-DD</lastmod>
     <changefreq>yearly</changefreq>
     <priority>0.8</priority>
   </url>
   ```

## Important Files

- `index.html` — landing page (DONE framework hero, `.pillars` grid, CTA). JSON-LD `Organization` + `WebSite`.
- `blog/index.html` — post listing (`.post-list` of `.post-card`) + JSON-LD `Blog`/`blogPost` array. Update when adding posts.
- `blog/beyond-vibe-coding-hve.html` — carries `<meta name="robots" content="noindex, nofollow">`; it is intentionally excluded from `sitemap.xml`. The other pages have no robots meta and are indexable.
- `styles.css` — `:root` tokens, header/nav/footer, landing sections (369 lines).
- `blog.css` — `.blog-post` typography, `.post-list`/`.post-card`, `.tag` pills (131 lines); requires `styles.css` first.
- `sitemap.xml` — hand-maintained; keep in sync with published (indexable) pages.
- `robots.txt` — `Allow: /` for all agents (explicitly welcomes GPTBot, ClaudeBot, PerplexityBot, etc.) and points to the sitemap.
- `llms.txt` — Markdown summary of the site/mission/posts for AI crawlers; update alongside new posts.
- `CNAME` — single line `otix.ai` (GitHub Pages custom domain).

## Runtime/Tooling Preferences

- **No runtime or package manager** for the site itself: no Node/`package.json`, no Bundler/Jekyll `_config.yml`, no Astro/etc. Do not introduce a build step or dependencies to solve an editing task — edit the HTML/CSS directly.
- Only tool needed for preview is a static file server (Python's `http.server` is fine).
- The `DONE framework` is a landing-page concept on `index.html`; there is no `done-framework.md` document to update.

## Testing & QA

There is no automated test suite, and none is expected. Verify changes by observation:

- Serve locally and load the affected page(s); confirm layout, links, and that `styles.css` + `blog.css` both applied.
- After adding/renaming a page, click through `nav`, the blog listing card, and the `blog-back` link to confirm relative paths resolve.
- Sanity-check the edited page parses as well-formed HTML and its JSON-LD block is valid JSON.
- `.DS_Store` is currently committed and there is no `.gitignore`; avoid adding new OS/editor cruft, and do not commit `.omc/` or `.omp/` state changes as part of site edits.
