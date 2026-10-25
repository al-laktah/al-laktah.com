# al-laktah.com

Source code for [al-laktah.com](https://al-laktah.com), the personal website of Mohammed Al-Laktah: about, CV, projects and blog.

## How it's built

- **[Hugo](https://gohugo.io)** turns plain Markdown files into a static site.
- **No theme, no npm, barely any JavaScript.** Hand-written templates and CSS, plus a few lines of script for the light/dark toggle.
- **[Catppuccin](https://catppuccin.com) colors:** Latte in light mode and Mocha in dark mode. The site follows the visitor's system setting, and a sun/moon button in the top bar overrides it. Code blocks use the matching Catppuccin highlighting styles.
- **GitHub Pages:** a GitHub Actions workflow builds the site and deploys it on every push to `main`.

## Repository layout

```
.
├── content/              Pages, written in Markdown
│   ├── _index.md         Home page intro (photo.jpg sits next to it)
│   ├── about.md
│   ├── projects/         One file per project
│   └── blog/             One file per post, or a folder for posts with images
├── layouts/              HTML templates
├── assets/css/           main.css (layout and colors) and syntax.css (generated code highlighting)
├── static/               Files served as-is: CVs, favicon
├── archetypes/           Front matter templates used by `hugo new`
├── hugo.toml             Site config: title, menu, social links
└── .github/workflows/    Build and deploy to GitHub Pages
```

## Running locally

Install [Hugo](https://gohugo.io/installation/) (the deploy workflow pins the exact version, see `HUGO_VERSION`), then:

```sh
hugo server -D               # live-reloading preview at http://localhost:1313, including drafts
hugo build --gc --minify     # production build into public/
```

## Content

Every Markdown file under `content/` becomes a page, with settings in its front matter:

| Field | Used by | Purpose |
| --- | --- | --- |
| `title`, `date` | all pages | Page title and publish date |
| `description` | all pages | Summary for search results and link previews, also shown in the project list |
| `draft: true` | all pages | Kept out of the production build |
| `stack`, `repo`, `website` | projects | Technologies used, source link and website link |
| `weight` | projects | Sort order; lower numbers come first |

`hugo new content blog/my-post.md` or `hugo new content projects/my-project.md` creates a new draft with the front matter ready to fill in.

## License

The templates and styles in `layouts/` and `assets/` are [MIT licensed](LICENSE), so feel free to reuse them. Everything else, including the writing, photo and CVs, is © Mohammed Al-Laktah and may not be reused without permission.
