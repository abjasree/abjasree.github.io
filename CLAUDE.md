# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Abjasree's portfolio, <https://abjasree.com>, in three designs. Vanilla HTML / CSS / JS built with Vite — no framework, no router. Each design is one `index.html` with its own `main.js` and `style.css`, and the code comments at the top of each module describe its behaviour in detail.

The repo is a fork of [souravas/souravas.github.io](https://github.com/souravas/souravas.github.io), which shares the designs and the build. Design and build changes come from there; when porting one, keep this site's content (name, roles, projects, contact details, analytics token, Formspree form, favicons) and leave out that site's personal pages.

```bash
npm install
npm run dev       # dev server on http://localhost:5173
npm run build     # production build → dist/
npm run preview   # preview the production build
```

## Deployment

- [.github/workflows/deploy.yml](.github/workflows/deploy.yml) builds pushes and pull requests to `main`, and deploys pushes (and *Run workflow*) to GitHub Pages with `actions/deploy-pages`. Pull requests only build. The upload sets `include-hidden-files: true`: without it `upload-pages-artifact` drops dot-folders, and `/.well-known/security.txt` never ships.
- [.github/workflows/rebuild.yml](.github/workflows/rebuild.yml) starts that workflow on the 1st of each month, so the build-time values roll over without a push. It is its own file because GitHub switches off a scheduled workflow after 60 days without repository activity, and that must never stop deploys on push.
- Dependabot opens weekly npm and Actions update PRs.

### Cloudflare

The domain is registered at Cloudflare and its DNS is there. The site's records (the apex's four GitHub Pages `A` and four `AAAA` records, and `www` as a `CNAME` to `abjasree.github.io`) are proxied, with SSL/TLS on Full (strict). GitHub keeps its own Let's Encrypt certificate behind the proxy; renewals get through because Cloudflare passes `/.well-known/acme-challenge/` to GitHub over plain HTTP. A Cloudflare 526 error means that certificate lapsed: set the records to DNS only until GitHub renews it. Mail to the domain runs through Cloudflare Email Routing (its MX and SPF records, and DMARC at `p=reject` reporting to Cloudflare's DMARC Management), so leave those records alone.

- **No HTML rewriting.** Pages must reach visitors byte for byte as GitHub serves them, because `csp-inline-hashes` pins every inline script and style. So these stay off: Email Address Obfuscation, Automatic HTTPS Rewrites, Rocket Loader, Bot Fight Mode (its JavaScript Detections inject an inline script the CSP blocks) and Web Analytics' automatic setup (the pages load the beacon themselves). To check, compare `curl -s https://abjasree.com/v1/` with `curl -s --resolve abjasree.com:443:185.199.108.153 https://abjasree.com/v1/`.
- **Headers**: the Managed Transform *Add security headers* adds `x-content-type-options`, `x-frame-options` and `referrer-policy`. GitHub's HSTS passes through.
- **Caching**: Browser Cache TTL respects GitHub's `max-age=600`. The cache rule *Hashed build files* gives `.js`, `.css` and `.woff2` under `/assets/` a year at the edge and in browsers. The cache rule *Fixed-name fonts and images* gives `/fonts/` and the `.webp` files in `/assets/` a year in browsers, while the edge keeps GitHub's 10 minutes. **So never replace one of those files in place**: save the new version under a new name and update every reference, including `profile.webp`, which is also the social image and is named in v1–v3's `og:image`, `twitter:image` and JSON-LD. The PDFs keep their names and their 10 minutes.
- **Redirects**: the redirect rule *CV to PDF* sends `/cv`, `/cv.html`, `/resume` and `/resume.html` to `/assets/resume.pdf` with a 301, so the `public/` stubs answer only if the rule is removed.
- Search, agent and training crawlers are all allowed, and Cloudflare leaves `robots.txt` as it is.

## Pages

| Route | Source | What it is |
|---|---|---|
| `/` | `v1/` | the default design; the `root-default` plugin copies built `v1/index.html` to `dist/index.html` (`DEFAULT_PAGES`) |
| `/v1`, `/v2`, `/v3` | `v1/`, `v2/`, `v3/` | the designs: Editorial, Systems Ledger, Control Room |
| `/cv`, `/resume` | `public/cv.html`, `public/resume.html` | 301 to `/assets/resume.pdf` from a Cloudflare redirect rule; the stubs meta-refresh there if the rule is gone |
| any unknown route | `public/404.html` | self-contained static 404 |
| `/sitemap.xml`, `/.well-known/security.txt` | build output | see Build |

`public/` is copied verbatim into the build: `CNAME`, favicons, manifest, `robots.txt`, the redirect stubs, `fonts/` and `assets/` (images, `resume.pdf`, `ms_thesis.pdf`). Browsers cache `fonts/` and the `.webp` images for a year, so a changed one needs a new file name (see Cloudflare).

## Build ([vite.config.js](vite.config.js))

Plugins, in order:

- `html-routes`: the dev and preview servers mirror GitHub Pages' routing (`/x` → `/x/index.html`, `/cv` → `/cv.html`).
- `build-stamps` (dev too): fills `__BUILD_DATE__`, `__BUILD_YEAR__`, and the whole years since `CAREER_START` (December 2021) as `__YEARS__`, `__YEARS_00__`, `__YEARS_WORD__`, `__YEARS_WORD_CAP__` and `__YEARS_ORDINAL_CAP__`. **Write these tokens, never the number**, wherever copy states the years. v3's hero uptime is also counted from December 2021, in its `main.js`.
- `json-ld-minify`, `inline-css` (each page's CSS bundle is inlined as a `<style>`, so there is no render-blocking stylesheet request), `csp-inline-hashes`, `html-minify`.
- `sitemap` (lists only `/`, with the build date as `lastmod`), `security-txt` (`Expires` 364 days out, since RFC 9116 wants under a year; contact `SECURITY_CONTACT`), `root-default`.

**CSP**: every page carries a CSP `<meta>`. At build time `csp-inline-hashes` swaps `'unsafe-inline'` for SHA-256 hashes of the page's inline `<script>`s and `<style>`s; dev keeps `'unsafe-inline'` so HMR works. Each design has an inline script, so its source has `script-src 'self' 'unsafe-inline'` for that swap. That literal `'self' 'unsafe-inline'` prefix is what the swap matches, so add other sources after it.

**Analytics**: the three designs load Cloudflare Web Analytics (`static.cloudflareinsights.com/beacon.min.js`, just before `</body>`) with this site's own token. Its CSP allows that host in `script-src` and `https://cloudflareinsights.com`, where the beacon reports, in `connect-src`.

## Designs (v1–v3)

Each design is self-contained; v2 and v3 bundle their own `fonts/`, and v1 uses `public/fonts/`. All three use `profile.webp` as the social image and name `/` as their canonical, so search engines index one copy of the résumé, while `og:url` stays per version. The footer's design switcher links the default as `/` and the others as `/vN/`. The contact forms post to Formspree and fall back to a `mailto:` link.
