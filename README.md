# al-laktah.com

My personal website, built with [Hugo](https://gohugo.io) from plain Markdown files and hosted on GitHub Pages.

## Day to day: editing content

Everything you write lives in `content/`. Each `.md` file becomes a page.

```
content/
├── _index.md          → home page intro
├── photo.jpg          → home page photo
├── about.md           → /about/
├── projects/
│   ├── _index.md      → /projects/ (intro text above the list)
│   └── my-project.md  → /projects/my-project/
└── blog/
    ├── _index.md      → /blog/ (intro text above the list)
    └── my-post.md     → /blog/my-post/
```

**Write a blog post:** create `content/blog/some-title.md`:

```markdown
---
title: "Some Title"
date: 2026-10-25
description: "One sentence, used in link previews and search results."
---

Your post in Markdown…
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

**Update the About page or home intro:** edit `about.md` or `_index.md`.

**Change the photo:** replace `content/photo.jpg`. Any size works since it's resized automatically; a portrait crop looks best. Delete it to hide the photo.

**Update the CV:** save the new export as `static/cv.pdf` (served at `/cv.pdf`). The *CV* menu link and the About page point straight at it. Keep file names in `static/` lowercase: Hugo lowercases menu links and GitHub Pages URLs are case-sensitive.

**Images:** turn the post into a folder and put images next to it:

```
content/blog/some-title/
├── index.md       ← the post itself
└── diagram.png    ← use ![Alt text](diagram.png) in index.md
```

Other site-wide files go in `static/` and are served as-is: `static/example.pdf` is at `/example.pdf`.

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
hugo new content blog/some-title.md
hugo new content projects/some-project.md
```

## How it's put together

| Path | What it is |
| --- | --- |
| `content/` | All pages (Markdown) |
| `hugo.toml` | Site title, social links, menu |
| `layouts/` | HTML templates (no third-party theme) |
| `assets/css/main.css` | The only stylesheet |
| `static/` | Files copied as-is (CV PDF, favicon, …) |
| `archetypes/` | Front-matter templates used by `hugo new` |
| `.github/workflows/deploy.yml` | Builds and deploys on every push to `main` |

There's no JavaScript, no `node_modules` and no theme to update. The only dependency is the Hugo version pinned in `deploy.yml`.

## One-time setup

1. **Push this repo to GitHub.**
2. **Enable Pages:** repo *Settings → Pages → Build and deployment → Source: **GitHub Actions***.
3. **Custom domain:** in the same Pages screen, set the custom domain to `al-laktah.com` and tick *Enforce HTTPS* once the certificate is issued.
4. **DNS** Set every record below to **Proxy status: DNS only** (grey cloud); GitHub can't verify the domain or issue the HTTPS certificate through Cloudflare's proxy:

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
