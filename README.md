# evanlih.photography

How this site is built, hosted and updated. This file lives at the repo root, so Hugo never publishes it to the site.

## How it's connected

```
this repo (GitHub, main branch)
   │  push
   ▼
GitHub Actions (.github/workflows/hugo.yaml)
   │  installs Hugo 0.140.1, pulls the Bridget theme, builds the site
   ▼
GitHub Pages (hosts the built files)
   ▲
   │  Cloudflare proxies visitors through to GitHub Pages
Cloudflare (DNS for evanlih.photography)
   ▲
visitors
```

- **Repo:** `EvanLih/evanlih.github.io`. Pushing to `main` is the only deploy step. The site is live a minute or two later.
- **Build:** [Hugo](https://gohugo.io) static site generator, run by the GitHub Actions workflow. Check the Actions tab on GitHub if a push doesn't show up.
- **Theme:** [Bridget](https://github.com/Sped0n/bridget), pulled in as a Hugo module (see `go.mod`). It isn't copied into the repo.
- **Domain:** `evanlih.photography` points at Cloudflare (its DNS answers are Cloudflare addresses), and Cloudflare forwards traffic to GitHub Pages. The custom domain is also set under the repo's Settings → Pages. DNS records live in the Cloudflare dashboard. Check there (or the registrar's site) to see where the domain itself is registered and when it renews.

## Folder map

| Path | What it is |
|---|---|
| `content/<tab>/` | One folder per tab: the photos plus a small `<tab>.md` file |
| `content/info/info.md` | The info page (gear, links, footer) |
| `config/_default/hugo.toml` | Site URL, site title, theme module |
| `config/_default/params.toml` | Description, image sizes, analytics settings |
| `layouts/` | Small overrides on top of the theme (the "open in browser" banner shown inside Instagram's in-app browser) |
| `static/` | Favicon files |
| `.github/workflows/hugo.yaml` | The build and deploy job |
| `public/`, `resources/` | Local build output. Ignored by git, safe to delete |

## Tabs

Each tab is a folder in `content/` with a `.md` file like this:

```yaml
---
type: _default
layout: single
url: /adventures/          # page address
menu:
  main:
    weight: 4              # position in the menu (lower = further left)
    identifier: 'adventures'
    title: 'adventures'    # text shown in the menu
unifiedAlt: '© Evan Lih'
_build:
  publishResources: false  # never publish the original photo files
---
```

Current tabs: places (home page, `url: /`), people, current, adventures, info.

**Add a tab:** copy one of the tab folders, rename it, change `url`, `identifier`, `title` and `weight`, swap in the photos.

**Rename a tab:** change the fields above and add the old address under `aliases:` so old links redirect (see `content/adventures/adventures.md`, which keeps `/rides/` working).

## Photos

- Photos show in filename order: `1.jpg`, `2.jpg`, `3.jpg`...
- JPG works best. Convert HEIC first.
- Hugo makes two resized WebP copies of each photo (800px preview, 2500px full view). Photos bigger than about 3000px on the long side just make the repo heavier and the build slower.
- **Strip location data before adding photos.** The repo is public, so anyone can download the originals and read GPS coordinates from them. The live site's resized copies don't carry it, but the repo files do. Photos exported from Instagram or with Location turned off on export are already clean.
- Instagram keeps full-resolution copies of carousel photos, which is a handy source for edited versions.

## Preview locally

Needs Hugo extended and Go installed (`brew install hugo go`):

```
hugo server
```

Then open http://localhost:1313. Edits reload automatically.

## Update the theme

```
hugo mod get -u github.com/Sped0n/bridget/v2
hugo mod tidy
```

Preview locally before pushing, and check the theme's release notes for breaking changes or a newer minimum Hugo version. If one is needed, bump `HUGO_VERSION` in `.github/workflows/hugo.yaml`.

## History

- 2026-11: added the adventures tab (Eureka to SF ride, from the Instagram post), updated Bridget to v2.3.1, fixed the Instagram link, removed GPS data from photos, cleaned up stray build files.
