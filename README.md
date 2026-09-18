# UBIKA service status page — preproduction

Repo for https://status.preprod.ubika.io

This is the preproduction copy of
[`service-status`](https://github.com/ubikasec/service-status): same theme,
layouts, build tooling and incident content, but its own `baseURL` and title.
Changes to the build (Hugo/cState upgrades, layout overrides, workflow) and new
incident posts are tried here first, then ported to production.

## RSS feed

Incidents are exposed as an RSS feed at
<https://status.preprod.ubika.io/index.xml> (one item per incident, template:
`layouts/index.xml`). Each item carries machine-readable `<category>` entries:
`status:ongoing|resolved`, `severity:<value>`, `type:informational`,
`affected:<component>` (one per component) and `resolvedWhen:<date>`, so that
feed consumers can filter on them without parsing the HTML description.

Internal e-mail notifications are sent from the production feed only, by a
flow that lives outside this repository. Use this repo to check the shape of
the feed before publishing on production.

## Local build

Hugo Extended is pinned in `.hugo-version` (shared with the GitHub Actions
workflow). The `Makefile` downloads that exact version into `.bin/` (ignored by
git), so no system-wide install is needed:

```sh
make serve   # live-reload dev server on http://127.0.0.1:1313/
make build   # production build into public/ (same flags as CI)
make check   # build + validate public/index.xml (RSS feed)
```

Requirements: `make`, `curl`, `tar`, `git`, `python3` (feed validation).

### Upgrading Hugo or the cState theme

1. Change the version in `.hugo-version` and/or `cd themes/cstate && git checkout <tag>`.
2. Run `make check` and compare the result with the current site.
3. Some files under `layouts/` are project-level copies of theme templates
   (each has a header comment explaining why): `index.xml` (RSS categories),
   `_default/small.html` (Hugo >= 0.146 template lookup) and
   `partials/index/summary.html` (RSS link fix, upstream PR #358). Diff them
   against `themes/cstate/layouts/` and port any upstream change, or delete
   them once the theme ships the fix.
   `partials/header.html` is a fourth copy, but it is **specific to this
   repo**: it adds the PREPRODUCTION badge displayed next to the logo. Keep it
   when upgrading the theme, and never port it to production.
4. Push a branch and open a pull request: the workflow builds the site without
   deploying it, and the `github-pages` artifact of the run can be downloaded
   and served locally as a preview. Deployment only happens on `main`.
