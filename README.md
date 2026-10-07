# github-infra

Declarative GitHub account wiring for johnsyweb. Today it manages **which repositories** the [Mend Renovate](https://github.com/apps/renovate) app can access.

Keeps fleet membership in git instead of the Mend UI, so adding or removing a Renovate repo is a reviewed PR that the sync workflow applies. Shared Renovate *config* and the no-App aube-lock workflow live in [johnsyweb/renovate-config](https://github.com/johnsyweb/renovate-config).

[![sync-renovate-repos](https://github.com/johnsyweb/github-infra/actions/workflows/sync-renovate-repos.yml/badge.svg)](https://github.com/johnsyweb/github-infra/actions/workflows/sync-renovate-repos.yml)

## Getting started

Edit [`renovate/repos.yaml`](renovate/repos.yaml) so `repos:` lists every repository the Mend app should access, then merge to `main` (or run **sync-renovate-repos** via `workflow_dispatch`). The workflow adds missing repos and removes extras.

```yaml
installation_id: 167440087
repos:
  - ambassy
  - progression
```

First-time account setup (Mend install, `installation_id`, `RENOVATE_SYNC_PAT`) is under [Local development](#local-development).

## Help

[GitHub Issues](https://github.com/johnsyweb/github-infra/issues). Preset and aube-lock behaviour: [johnsyweb/renovate-config](https://github.com/johnsyweb/renovate-config).

## Maintainers

[johnsyweb](https://github.com/johnsyweb) (Pete Johns).

## Development status

Maintained. Renovate membership is the first wiring surface; other account automation may land later.

## Local development

Requires `gh`, `python3`, and a classic PAT exported as `RENOVATE_SYNC_PAT` (`repo` + `read:user`):

```bash
export RENOVATE_SYNC_PAT=…   # classic: repo + read:user
./script/sync-renovate-repos
```

### One-time: install Mend Renovate

1. Open [github.com/apps/renovate](https://github.com/apps/renovate) → **Configure** / **Install**.
2. Choose **Only select repositories**.
3. Select at least one repo (e.g. ambassy) if GitHub requires it, or leave empty if the API will add repos.
4. Save.

### Find `installation_id`

`GET /user/installations` only accepts a **GitHub App user-to-server** token, so a normal `gh auth` / PAT call returns HTTP 403. Read the id from the browser:

1. Open [Installed GitHub Apps](https://github.com/settings/installations) (or [Configure Renovate](https://github.com/apps/renovate)).
2. Click **Configure** next to Renovate.
3. The URL is `https://github.com/settings/installations/<installation_id>` — put that number in `renovate/repos.yaml`.

### Create `RENOVATE_SYNC_PAT`

| Scope | Why |
| --- | --- |
| `repo` | Add/remove repositories on the installation |
| `read:user` | List repositories already on the installation |

Create at [github.com/settings/tokens](https://github.com/settings/tokens) (classic), then:

```bash
gh secret set RENOVATE_SYNC_PAT --repo johnsyweb/github-infra
```

Rotate periodically. If sync fails with HTTP 403 mentioning `read:user`, recreate the PAT with both scopes and set the secret again.

### Adding a repo to the Renovate fleet

1. Land Renovate + aube-lock in that repository, including the README refresh gate (see [renovate-config — Use in a repo](https://github.com/johnsyweb/renovate-config#use-in-a-repo)).
2. Add the repo name under `repos:` in `renovate/repos.yaml`.
3. Merge; sync workflow updates the Mend installation.

### Out of scope

- OpenTofu / Terraform state
- Creating the Mend app itself (marketplace consent only)
- Per-repo aube-lock GitHub Apps (not used — see renovate-config no-App pattern)

## Contributing

Open a PR that updates `renovate/repos.yaml` (and docs when the checklist changes). Prefer Conventional Commits. After merge, confirm the [sync-renovate-repos](https://github.com/johnsyweb/github-infra/actions/workflows/sync-renovate-repos.yml) workflow succeeded.

## Releasing

Pushing to `main` runs **sync-renovate-repos**, which reconciles Mend installation membership to `renovate/repos.yaml`. Manual runs use **Actions → sync-renovate-repos → Run workflow**.
