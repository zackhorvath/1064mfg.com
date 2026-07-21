# 1064mfg.com

Marketing site for **1064 MFG LLC** — 3D printing and laser engraving
services. Built with [Hugo](https://gohugo.io), no external theme; a small
hand-rolled "cassette futurism" design system lives in `assets/css/main.css`.
Deployed to GitHub Pages via GitHub Actions.

## Stack

- Hugo (tested against `v0.164.0`, extended edition — needed for asset
  minification/fingerprinting)
- No theme / Hugo Modules — layouts and CSS are all in this repo
- GitHub Actions → GitHub Pages (official `actions/deploy-pages` flow, not a
  `gh-pages` branch)

## Local development

### Option A — Docker (no local Hugo install needed)

```sh
docker run --rm -it -v "$(pwd)":/src -p 1313:1313 hugomods/hugo:exts \
  server --bind 0.0.0.0
```

Then open http://localhost:1313.

### Option B — Local Hugo binary

Install the **extended** version of Hugo (required — the build uses
`resources.Minify`/`fingerprint`, which are extended-only features):

```sh
hugo server -D
```

### Build

```sh
hugo --minify --gc
```

Output goes to `public/`.

## Site structure

```
hugo.toml                  Site config, menu, global params (email, socials)
content/_index.md          Home page copy: hero, services, portfolio, process
content/contact.md         Contact page copy + Formspree form ID
layouts/                   Hand-rolled templates (no theme)
assets/css/main.css        Design system (colors, type, components)
layouts/partials/icon.html Inline SVG icon set used on the portfolio grid
static/CNAME                Custom domain for GitHub Pages
static/favicon.svg
```

There's intentionally no blog/post section — this is a two-page site (Home,
Contact) by design. If that changes later, add a `content/post/` section and
a matching `layouts/_default/list.html` / `single.html`.

## Editing content

Almost everything on the home page (hero copy, the two service blocks, the
portfolio grid, the process steps) is data in the front matter of
`content/_index.md` — edit the YAML there rather than the template. Same for
`content/contact.md`.

- **Portfolio items** (`work` list in `content/_index.md`): each entry is
  `code`, `title`, `tag`, `icon`. `icon` must be one of `printer`, `laser`,
  `gear`, `plaque` (defined in `layouts/partials/icon.html`) — add new ones
  there if you need more variety, or swap the icon block for real photos
  once you have them (`work-card__icon` in the CSS expects a square image).
- **Contact form**: create a form at [formspree.io](https://formspree.io) (or
  swap in any other static-form backend) and replace
  `REPLACE_WITH_FORMSPREE_ID` in `content/contact.md`. Until that's set, the
  page shows a visible on-page notice and the mailto fallback still works.
- **Company email / socials**: `[params]` in `hugo.toml`.

## Deployment

Push to `main` — `.github/workflows/deploy.yml` builds with Hugo and
publishes via GitHub's official Pages deployment (`actions/configure-pages`
+ `upload-pages-artifact` + `deploy-pages`).

One-time repo setup required:

1. **Settings → Pages → Build and deployment → Source**: set to
   **GitHub Actions** (not "Deploy from a branch").
2. **Settings → Pages → Custom domain**: set to `1064mfg.com` (this also
   regenerates/verifies the `CNAME` file expectation — the repo already
   ships `static/CNAME` so the workflow keeps it on every deploy).
3. **DNS**, at your domain registrar:
   - Apex domain (`1064mfg.com`): four `A` records pointing at GitHub Pages'
     IPs:
     ```
     185.199.108.153
     185.199.109.153
     185.199.110.153
     185.199.111.153
     ```
   - `www.1064mfg.com` (optional): `CNAME` record to `zackhorvath.github.io`.
   - Enable **Enforce HTTPS** in the Pages settings once DNS has propagated
     and GitHub has issued a certificate (can take a few minutes to an hour).

After that, every push to `main` redeploys automatically.

## Design system notes

- Palette, type, and component tokens are all CSS custom properties at the
  top of `assets/css/main.css` (`--bg`, `--accent`, `--mono`, etc.) — change
  them there rather than hunting through individual rules.
- Typography is a system monospace stack (no webfont download) to keep the
  terminal/industrial feel without adding an external font dependency. Swap
  `--mono` for a self-hosted `@font-face` (e.g. a licensed Space Mono /
  JetBrains Mono woff2) if you want a more distinctive look later.
- Portfolio icons are inline SVG (`layouts/partials/icon.html`) so they
  inherit `currentColor` and need no extra HTTP requests.
