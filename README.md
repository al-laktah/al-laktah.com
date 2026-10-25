# al-laktah.com

My personal website, built with [Hugo](https://gohugo.io) from plain Markdown files and hosted on GitHub Pages.

## Day to day: editing content

Everything you write lives in `content/`. Each `.md` file becomes a page.

```
content/
├── _index.md          → home page intro
├── about.md           → /about/
├── cv.md              → /cv/
├── projects/
│   ├── _index.md      → /projects/ (intro text above the list)
│   └── my-project.md  → /projects/my-project/
└── articles/
    ├── _index.md      → /articles/ (intro text above the list)
    └── my-post.md     → /articles/my-post/
```

**Write a new article:** create `content/articles/some-title.md`:

```markdown
---
title: "Some Title"
date: 2026-10-25
description: "One sentence, used in link previews and search results."
---

Your article in Markdown…
```

**Add a project:** create `content/projects/some-project.md`:

```markdown
---
title: "Some Project"
date: 2026-10-25
description: "One-line summary shown in the project list."
stack: ["Python", "AWS"]       # optional
repo: "https://github.com/…"   # optional
url: "https://…"               # optional
weight: 1                      # optional: lower numbers are listed first
---

Longer write-up…
```

**Update the CV, About page or home intro:** edit `cv.md`, `about.md` or `_index.md`.

**Images:** turn the post into a folder and put images next to it:

```
content/articles/some-title/
├── index.md       ← the post itself
└── diagram.png    ← use ![Alt text](diagram.png) in index.md
```

Site-wide files (e.g. `cv.pdf`) go in `static/`; `static/cv.pdf` is served at `/cv.pdf`.

**Drafts:** add `draft: true` to the front matter to keep a page off the live site.

**Social links and menu:** edit `hugo.toml`. These rarely change.

Commit and push to `main`, and the site rebuilds and goes live in about a minute.

## Preview locally

```sh
brew install hugo      # once
hugo server -D         # http://localhost:1313, live-reloads; -D includes drafts
```

Optional shortcut that creates a file with the front matter pre-filled (as a draft):

```sh
hugo new content articles/some-title.md
hugo new content projects/some-project.md
```

## How it's put together

| Path | What it is |
| --- | --- |
| `content/` | All pages (Markdown) |
| `hugo.toml` | Site title, social links, menu |
| `layouts/` | HTML templates (no third-party theme) |
| `assets/css/main.css` | The only stylesheet |
| `static/` | Files copied as-is (favicon, PDFs, …) |
| `archetypes/` | Front-matter templates used by `hugo new` |
| `.github/workflows/deploy.yml` | Builds and deploys on every push to `main` |

There's no JavaScript, no `node_modules` and no theme to update. The only dependency is the Hugo version pinned in `deploy.yml`.

## One-time setup

1. **Push this repo to GitHub.**
2. **Enable Pages:** repo *Settings → Pages → Build and deployment → Source: **GitHub Actions***.
3. **Custom domain:** in the same Pages screen, set the custom domain to `al-laktah.com` and tick *Enforce HTTPS* once the certificate is issued.
4. **DNS** at your registrar/DNS provider (keep your existing MX/email records untouched):

   | Type | Name | Value |
   | --- | --- | --- |
   | A | `@` | `185.199.108.153` |
   | A | `@` | `185.199.109.153` |
   | A | `@` | `185.199.110.153` |
   | A | `@` | `185.199.111.153` |
   | AAAA | `@` | `2606:50c0:8000::153` |
   | AAAA | `@` | `2606:50c0:8001::153` |
   | AAAA | `@` | `2606:50c0:8002::153` |
   | AAAA | `@` | `2606:50c0:8003::153` |
   | CNAME | `www` | `<your-github-username>.github.io` |

5. **Verify the domain** in your GitHub account settings (*Settings → Pages → Verified domains*) so no one else can claim it.
