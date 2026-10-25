---
title: "al-laktah.com"
date: 2026-10-25
description: "This website: a personal site built with Hugo from plain Markdown, themed with Catppuccin and deployed to GitHub Pages."
stack: ["Hugo", "CSS", "GitHub Actions", "GitHub Pages", "Cloudflare DNS"]
repo: "https://github.com/al-laktah/al-laktah.com"
website: "https://al-laktah.com"
---

The site you're reading. I wanted a personal site that is simple to maintain: every page is a Markdown file, and publishing is a `git push`.

## How it works

- **Content in Markdown.** The About page, projects and blog posts are plain Markdown files with a little front matter. Hugo turns them into static HTML.
- **No theme or framework.** A few hand-written templates and CSS, so there are no theme updates or npm dependencies to keep up with. The only JavaScript is a few lines for the light/dark toggle.
- **Catppuccin colors.** Latte in light mode and Mocha in dark mode, following the system setting unless you pick one with the toggle. Code blocks use the matching Catppuccin highlighting.
- **Automated deploys.** A GitHub Actions workflow builds the site with a pinned Hugo version and publishes it to GitHub Pages on every push to `main`.
- **Custom domain.** DNS is hosted on Cloudflare and set to DNS only (not proxied), so GitHub Pages can verify the domain and issue its HTTPS certificate.

The templates and styles are MIT licensed, so feel free to reuse them.
