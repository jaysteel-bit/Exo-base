# Exo Base — rebrand notes

Exo Base is Exo Enterprise's Airtable alternative, forked from
[baserow/baserow](https://github.com/baserow/baserow) (MIT). It runs as a
Flow OS surface — the Exo Base tab in Launch and the third tab in Team — and
as a standalone site at `https://base.exoflowos.com`.

## What this fork changes (v1, 2026-11-18)

Branding only. No behaviour, no schema, no licensing changes:

| File | Change |
|------|--------|
| `web-frontend/modules/core/static/img/logo.svg` | Baserow wordmark → "Exo Base" wordmark with the Exo orb |
| `web-frontend/modules/core/static/img/logoOnly.svg` | Baserow mark → Exo orb |
| `web-frontend/modules/core/static/img/favicon_{16,32,48,192}.png` | Regenerated as the Exo orb (rim-lit so it survives dark and light) |
| `web-frontend/modules/core/head.js` | `title` / `titleTemplate` → `Exo Base` |
| `web-frontend/modules/core/components/ExternalLinkBaserowLogo.vue` | Bottom-left logo link → `https://exoflowos.com` |

The component keeps its upstream name (`ExternalLinkBaserowLogo`) so upstream
merges stay clean. Rename it only if we stop merging from upstream.

## Deployment

`/opt/exo-base/compose.yml` on the Contabo box (compute barn). Single
all-in-one container, loopback `127.0.0.1:9180`, plain HTTP inside; TLS ends at
Cloudflare. Public host reaches it through the `exo-contabo` tunnel
(`base.exoflowos.com → http://127.0.0.1:9180`, tunnel config v7).

### Build from the release branch, not `develop` (2026-11-18)

The all-in-one image composes our frontend with **`baserow/backend:<version>` from Docker Hub**, i.e. the released
backend. So the rebrand must sit on **the release commit**, not on post-release `develop`:

- Branch `exo-base-2.4.3` = upstream tag `2.4.3` (`d897b90`) + the rebrand commits, cherry-picked.
- The workflow asserts the branch name is `exo-base-<version>` and that
  `web-frontend/package.json` matches, and fails otherwise.

Why the guard exists: the first attempt put the rebrand on `develop` (`74d6625`), which is *after* the 2.4.3 release.
A newer frontend against the released backend still reports version 2.4.3 on both sides, so the mismatch is invisible
until something behaves oddly. Claude caught it before CI was ever run. When bumping Baserow: fetch the new tag, branch
`exo-base-<new version>` from it, cherry-pick the rebrand commits onto it, dispatch the workflow on that branch.

Low-memory mode is on (`BASEROW_RUN_MINIMAL=yes`, `BASEROW_AMOUNT_OF_WORKERS=1`)
because the box shares RAM with the software factory and Hermes. Steady state is
~1.7 GB.

Public signups are **off** (`allow_new_signups=false`). The first account
(`admin@exoflowos.com`) is the instance admin; credentials live in the
workspace's key store, not in this repo.

## License boundary — read before touching `premium/` or `enterprise/`

The repository root is MIT. `premium/` and `enterprise/` are **not**: they are
source-available under Baserow's commercial licence. Kanban, Calendar, Timeline,
Comments, row/field-level permissions, SSO and audit log live there.

Copying that code into our build is a licence violation, not a fork decision.
Anything we want from those directories has to be either (a) licensed, or
(b) written by us from scratch against the MIT core's public APIs. Decide
deliberately, in writing, before any of it lands in this fork.

## Sign-in plan (in progress)

Team members will not get a second login. Flow OS holds the Clerk session; a
bridge endpoint on this instance exchanges it for a Baserow session cookie
(`jwt_token`), so the pane opens already signed in, and we can map Clerk orgs to
Baserow workspaces for department scoping later.
