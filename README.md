# dashboard-site

Builds and publishes [getdashboard.dev](https://getdashboard.dev/), the documentation of
[Dashboard](https://github.com/stefanrenne/dashboard) and its
[widgets collection](https://github.com/stefanrenne/dashboard-widgets). The site is the `gh-pages`
branch of this repository; GitHub Pages serves it as it is. Nothing here is written by hand: the
pages come from the `docs/` folders of those two repositories and a page per widget.

The site has two parts, each with its own releases and its own version menu:

| Part | Latest | Develop | From |
|---|---|---|---|
| Dashboard | `/` | `/develop/` | dashboard's `docs/` |
| Widgets | `/widgets/` | `/widgets/develop/` | the collection's `docs/` and `widgets/<id>/` |

`/version.json` has the latest version of each part, for the menus; `robots.txt` keeps the develop
versions out of search engines.

## How it is built

[`.github/workflows/site.yml`](.github/workflows/site.yml), started by the other two repositories
(with a fine-grained token in their secret `PAGES_DISPATCH_TOKEN`), or by hand:

| Mode | Started by | Does |
|---|---|---|
| `dashboard-develop` | a push to dashboard's `develop` | builds `/develop/` again |
| `dashboard-release` | a push to dashboard's `main` | builds `/develop/` again, then **copies** it to `/` |
| `widgets-develop` | a push to the collection's `develop` (docs, widgets, its mock) | builds `/widgets/develop/` again |
| `widgets-release` | a release of the collection | builds `/widgets/develop/` again, then **copies** it to `/widgets/` |
| `bootstrap` | by hand | all four: a whole site, the latest parts from dashboard's `main` and the newest collection release |

A release is a copy of its part's develop site, and leaves the other part alone: what was on the
develop address is what is released. The pages do not depend on the version they were built for
(the menus read it from the address), and dashboard's `scripts/pages-promote.sh` points the base
path and the links to the `develop` branch at the latest address and `main`.

Each widget image has a `.hash` next to it: its folder in the collection, the collection's mock and
what draws it in the dashboard. A build takes a picture only of a widget whose hash changed, and
starts Playwright only when one did.

The details: [Publishing](https://getdashboard.dev/extend/contributing/publishing.html#documentation-site).
