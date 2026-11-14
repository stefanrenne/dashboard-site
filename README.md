# dashboard-site

Builds and publishes [getdashboard.dev](https://getdashboard.dev/), the documentation of
[Dashboard](https://github.com/stefanrenne/dashboard). The site is the `gh-pages` branch of this
repository; GitHub Pages serves it as it is. Nothing here is written by hand: the docs are
`docs/` in the dashboard repository, the widget pages come from the
[widgets collection](https://github.com/stefanrenne/dashboard-widgets).

| Where | What |
|---|---|
| `/` | The release: dashboard's `main`, the collection's newest release |
| `/develop/` | What is on dashboard's `develop` and the collection's `develop` (a banner says so; `robots.txt` keeps it out of search engines) |
| `/version.json` | The release's version, for the version menu |

## How it is built

[`.github/workflows/site.yml`](.github/workflows/site.yml), started by the other two repositories
(with a fine-grained token in their secret `PAGES_DISPATCH_TOKEN`), or by hand:

| Mode | Started by | Does |
|---|---|---|
| `develop` | a push to dashboard's `develop` | builds `/develop/` again |
| `release` | a push to dashboard's `main` | builds `/develop/` again, then **copies** it to `/` |
| `collection` | a release of the widgets collection | builds `/` again from `main`'s docs and that release |
| `bootstrap` | by hand | `develop`, then `collection`: a whole site |

A release is a copy: what was on `/develop/` is what is released. The pages do not depend on the
version they were built for (the menu reads it from the address), and dashboard's
`scripts/pages-promote.sh` points the base path and the links to the `develop` branch at the root
and `main`.

Each widget image has a `.hash` next to it: its folder in the collection, the collection's mock and
what draws it in the dashboard. A build takes a picture only of a widget whose hash changed, and
starts Playwright only when one did.

The details: [Publishing](https://getdashboard.dev/extend/contributing/publishing.html#documentation-site).
