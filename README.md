# cochoy.fr

Personal website of Jeremy Cochoy — portfolio, blog, research, teaching materials, and music.
Built with Jekyll and hosted on GitHub Pages at [www.cochoy.fr](https://www.cochoy.fr/).

## Tech stack

- **Static site generator:** Jekyll 4.4
- **Theme:** Custom layouts based on Hyde/Poole, with minima as the base gem theme
- **CSS frameworks:** Bootstrap 3 (bundled in `public/bootstrap/`), Hyde, Poole
- **Markdown engine:** Kramdown
- **Math rendering:** MathJax (loaded from CDN)
- **Syntax highlighting:** Rouge (with `public/css/syntax.css`)
- **Deployment tool:** `jgd` (Jekyll GitHub Deploy)

## Repository structure

```
_config.yml          # Jekyll configuration
_includes/           # Reusable HTML partials (head, navbar, footer, etc.)
_layouts/            # Page templates (default, home, page, post)
_posts/              # Blog posts, organized in subdirectories with data/ assets
assets/              # Minima theme assets (SVG icons)
blog/                # Blog index page
public/              # Static assets served as-is
  bootstrap/         #   Bootstrap 3 CSS, JS, fonts (Glyphicons)
  css/               #   Hyde, Poole, syntax highlight stylesheets
  favicon.ico        #   Favicon
  style.css          #   Custom site-wide styles
research/            # Research page and publications
resume/              # CV in HTML, PDF, TeX, DOCX formats
teaching/            # Teaching materials (Android lessons as git submodule)
talks/               # Conference talks
music/               # Music section
pdfs/                # Hosted PDF documents (Perlin noise, periodic functions)
gb-doc/              # Game Boy documentation
hyper-flop/          # Hyper-flop web app
agreg-dev/           # Agreg development notes
CNAME                # Custom domain: www.cochoy.fr
Gemfile              # Ruby dependencies
Gemfile.lock         # Locked dependency versions
```

## Blog posts

Posts live in `_posts/<slug>/` subdirectories. Each can contain a `data/` folder with
images, audio, archives, etc. that are referenced with relative paths (e.g. `data/image.png`).
The `jekyll-postfiles` plugin copies these assets to the built output.

## Branch workflow

| Branch | Purpose |
|---|---|
| `development` | Active development — source Markdown, layouts, config |
| `master` | Built static HTML — served by GitHub Pages |

**Never edit `master` directly.** It is overwritten by the deploy tool.

## Local development

```bash
# Install dependencies
bundle install

# Serve locally with live reload
bundle exec jekyll serve
# Site available at http://127.0.0.1:4000/

# Build without serving
bundle exec jekyll build
# Output goes to _site/
```

## Deployment

Deploy the `development` branch to `master` (GitHub Pages):

```bash
bundle exec jgd -r development -b master
```

This builds the site from `development` and force-pushes the result to `master`.

## Plugins

- **jekyll-feed** — Generates Atom feed at `/feed.xml`
- **jekyll-paginate** — Blog pagination (10 posts per page)
- **jekyll-redirect-from** — URL redirects via front matter
- **jekyll-postfiles** — Copies non-Markdown files from post directories to output
