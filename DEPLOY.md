# Deploy (NAUCode copy)

Read this before pushing to `master`, releasing, or moving this repo. Facts
checked on 2026-10-20 unless marked "chưa kiểm" (not verified). This file is
about NAUCode's copy of the repo; upstream Paperclip's own deploy docs are in
`doc/` and `docs/deploy/`.

## Repo

| | |
| --- | --- |
| Current location | `github.com/naucodehq/paperclip` (public) |
| Previous location | `github.com/2ngnhan/paperclip` (GitHub redirects it) |
| Status | Transferred to `naucodehq` on 20/10/2026 (đã chuyển 20/10/2026) |
| Default branch | `master` |
| Upstream | `github.com/paperclipai/paperclip` (remote `upstream` in `~/dev/paperclip`). GitHub does not list this repo as a fork of it. |
| Local clone | `~/dev/paperclip`, `origin` already points at `naucodehq/paperclip` |

This is the **legacy fat fork** behind n'clerk. Production does not run it:
the n'clerk image (repo `n-clerk`, Railway, app.nclerk.com) clones upstream
`paperclipai/paperclip` at a pinned ref. The plan to retire this fork is
`n-clerk/docs/fork-thinning.md`.

## Deploy targets

| Target | State on 2026-10-20 |
| --- | --- |
| npm canary (`paperclipai`, `@paperclipai/*`) via `.github/workflows/release.yml`, environment `npm-canary` | Not live. Every push to `master` runs `verify_canary`, then `publish_canary` with npm trusted publishing (OIDC). The last runs (2026-03-24, 2026-07-10) failed in `verify_canary` (tests). The two `npm-canary` deployments of 2026-03-22 failed at publish. Per `doc/RELEASE-AUTOMATION-SETUP.md`, npm trusted publishing is set up for `paperclipai/paperclip`, not for this repo (chưa kiểm on npm). |
| npm stable, environment `npm-stable` | Manual `workflow_dispatch` only. Never used here. |
| Cloudflare Pages project `ppclip` | Was connected in March 2026 (split-hosting experiment, check run on `f59ad00`). No Cloudflare check run on the latest `master` commit, so probably disconnected (chưa kiểm on the dashboard). |
| Render | Mentioned in commit `1a2c945` ("Render-compatible"). No status or deployment from Render on GitHub (chưa kiểm). |

## How a deploy happens

There is no wanted deploy from this repo. What a push does today:

- Push to `master`: `Release` (`verify_canary` then `publish_canary`) and `Refresh Lockfile` run. If tests ever pass, `publish_canary` tries to publish to npm. Do not let that happen from this copy.
- Pull request to `master`: `PR` workflow (policy, typecheck, tests). No publish.
- `e2e.yml` and `release-smoke.yml`: manual only.

Recommended before anyone merges to `master` again: disable the Release
workflow on this repo (`gh workflow disable Release -R naucodehq/paperclip`),
or archive the repo as `fork-thinning.md` plans. Not done; it is a settings
change for the owner.

## Manual deploy

None. To run the code locally:

```sh
pnpm install
pnpm dev        # embedded PGlite when DATABASE_URL is unset, see AGENTS.md
```

## Verify

- `gh run list -R naucodehq/paperclip -L 5`: after a merge, confirm `publish_canary` did not succeed.
- `npm view paperclipai dist-tags` must not show a version published from this repo.

## Rollback

Nothing deploys, so nothing to roll back. If a canary ever gets published by
mistake: `npm deprecate` that version (owner of the npm package required),
then disable the workflow.

## Secrets and vars (names only)

- Repo Actions secrets: none listed. `e2e.yml` reads `ANTHROPIC_API_KEY` (not set).
- Environments: `npm-canary`, `npm-stable` (OIDC trusted publishing, no token).
- Runtime env for the app is upstream's: see `docs/deploy/environment-variables.md`.

## After the transfer to naucodehq (done 20/10/2026)

- [x] Hardcoded `2ngnhan/paperclip` strings in this repo: none.
- [x] `~/dev/paperclip` `origin` points at `naucodehq/paperclip`.
- [x] `n-clerk/docs/fork-thinning.md` updated to `naucodehq/paperclip`.
- [ ] Decide: disable `Release`, or archive the repo (see above).
- [ ] If a Cloudflare Pages project `ppclip` or a Render service still points at this repo, delete it or reconnect it through the Cloudflare / Render GitHub app on `naucodehq` (chưa kiểm).
- [ ] Upstream PRs from this repo: since it is not a GitHub fork of `paperclipai/paperclip`, a cross-repo PR needs a real fork as the head.
