# Abjasree's Portfolio

Source for [abjasree.com](https://abjasree.com). Vanilla HTML / CSS / JS built with Vite — no framework. GitHub Actions builds and deploys to GitHub Pages on every push to `main`.

## Getting Started

```bash
npm install
npm run dev       # dev server on http://localhost:5173
npm run build     # production build → dist/
npm run preview   # preview the production build
```

## Deployment

Pushes to `main` are built and deployed automatically via [.github/workflows/deploy.yml](.github/workflows/deploy.yml) using `actions/deploy-pages`. Pull requests run the build only. To trigger a deploy manually, use *Run workflow* on the Actions tab. [.github/workflows/rebuild.yml](.github/workflows/rebuild.yml) redeploys on the 1st of each month, so the values filled in at build time (years of experience, the `security.txt` expiry) stay current without a push.

The domain and its DNS are on Cloudflare, which proxies the site in front of GitHub Pages: it adds security headers, caches the fonts and images for a year, and redirects `/cv` and `/resume`. It is set not to rewrite any HTML, since each page's CSP pins its inline scripts and styles by hash. [CLAUDE.md](CLAUDE.md#cloudflare) lists the settings.

## URLs

- Local dev: <http://localhost:5173>
- Production: <https://abjasree.com>
- Page versions: `/` serves v1 by default; `/v1`, `/v2`, and `/v3` open each design explicitly, and each names `/` as its canonical
- Resume redirects: `/cv`, `/resume` → `/assets/resume.pdf`
- Sitemap: `/sitemap.xml`, generated at build time
- Security contact: `/.well-known/security.txt`, generated at build time

## Project Structure

```
abjasree.github.io/
├── v1/                     # "Editorial" design, the default (also served at /)
│   ├── index.html
│   ├── main.js
│   └── style.css
├── v2/                     # "Systems Ledger" design, served at /v2
│   ├── index.html
│   ├── main.js
│   ├── style.css
│   └── fonts/
├── v3/                     # "Control Room" design, served at /v3
│   ├── index.html
│   ├── main.js
│   ├── style.css
│   └── fonts/
├── public/                 # copied verbatim into the build
│   ├── CNAME               # custom domain
│   ├── 404.html            # static 404 served by GitHub Pages
│   ├── favicon.svg         # site icon; favicon.ico and the PNGs are renders of it
│   ├── manifest.webmanifest
│   ├── robots.txt
│   ├── cv.html, resume.html  # meta-refresh → /assets/resume.pdf
│   ├── fonts/              # v1's fonts, also used by 404.html
│   └── assets/             # images, resume.pdf, ms_thesis.pdf
├── vite.config.js          # multi-page build, default-page copy, CSS inlining,
│                           # CSP hashing, HTML minify, build-time values
│                           # (years, sitemap, security.txt), dev routes
└── .github/
    ├── dependabot.yml      # weekly npm and Actions updates
    └── workflows/
        ├── deploy.yml      # build, and deploy on push to main
        └── rebuild.yml     # monthly redeploy for the build-time values
```

## Links

- Live site: <https://abjasree.com>
- Repository: <https://github.com/abjasree/abjasree.github.io>
