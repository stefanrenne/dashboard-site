# To do

Open work. Remove an item when it is done.

## Security review (2026-11-15)

This repository holds no secrets of its own: `site.yml` runs with the workflow's `GITHUB_TOKEN`
(`contents: write`, so it can push `gh-pages`, which is the public site). The two other repositories
start it with `PAGES_DISPATCH_TOKEN` (Actions only, on this repository). What an attacker would want
here is that token, to change getdashboard.dev. Sound already: modes are a fixed choice, the inputs
reach the shell through environment variables, `gh-pages` is checked out without the code of the
other repositories, and CNAME is left to GitHub.

- [ ] **MEDIUM: third-party code runs where it can read the push token.** "The published site"
  writes `GITHUB_TOKEN` into `site/.git/config` (the remote URL) before "Stage", and Stage mounts
  the whole workspace into Playwright's container, where `npm install playwright@<newest>` and the
  widgets' code run. Anything in there can read the token and push to `gh-pages`. Fix: check out
  `gh-pages` without credentials (the repository is public) and give the token to the push alone,
  in Publish (`git -c http.extraheader="AUTHORIZATION: basic …" push`, or the URL with the token
  only in that command); mount only the folders the container needs.
- [ ] **LOW: the newest Playwright from npm, not a pinned one.** `npm view playwright version` picks
  whatever was published last. Pin the version in the workflow (and the image tag with it), and let
  Dependabot bump it (below), so a bad release does not reach the build the day it appears.
- [ ] **LOW: a collection tag reaches a shell command as text.** Stage builds `cmd` as a string
  with `'$4'` (the collection's tag) and paths inside single quotes, run by `sh -c`. A git tag may
  hold a `'`; a tag such as `v1'$(…)'` would run in the build. Whoever can make that tag can already
  release code to every installation, so the gain is small, but pass the values as environment
  variables to the container (`-e TAG -e CACHE`) and quote them there, instead of building a string.
- [ ] **LOW: no Dependabot here.** The actions (`actions/checkout`, `actions/setup-node`,
  `actions/jekyll-build-pages`) are on major tags that nothing updates. Add `.github/dependabot.yml`
  for github-actions (weekly, no auto-merge), and pin the actions to commit SHAs as in the dashboard
  repository's todo.
- [ ] **LOW: protect the branches.** A ruleset on `main` (the workflow: owner only, no force push) and
  on `gh-pages` (only the workflow's pushes and the owner; no deletion), so a leaked token with
  `contents` on this repository cannot replace the site's history unnoticed.
