# johnsyweb/github-infra

Declarative GitHub account wiring for johnsyweb. Today this manages **which repositories** the [Mend Renovate](https://github.com/apps/renovate) app can access.

Shared Renovate *config* and the no-App aube-lock workflow live in [johnsyweb/renovate-config](https://github.com/johnsyweb/renovate-config).

## One-time: install Mend Renovate

1. Open [github.com/apps/renovate](https://github.com/apps/renovate) → **Configure** / **Install**.
2. Choose **Only select repositories**.
3. Select **ambassy** (or leave empty if the API will add it — GitHub may require at least one; pick ambassy).
4. Save.

### Find `installation_id`

```bash
gh api user/installations --jq '.installations[] | select(.app_slug=="renovate") | {id, repository_selection, html_url}'
```

Put that `id` into [`renovate/repos.yaml`](renovate/repos.yaml) as `installation_id`.

### Create `RENOVATE_SYNC_PAT`

GitHub’s “add/remove repo on installation” endpoints require a **classic** PAT with the **`repo`** scope (not a fine-grained token, not `GITHUB_TOKEN`).

```bash
# Create a classic PAT in the browser (repo scope), then:
gh secret set RENOVATE_SYNC_PAT --repo johnsyweb/github-infra
```

Paste the token when prompted. Rotate periodically.

## Desired state

[`renovate/repos.yaml`](renovate/repos.yaml):

```yaml
installation_id: 12345678
repos:
  - ambassy
```

Push to `main` (or run **sync-renovate-repos** via `workflow_dispatch`) to apply. The workflow adds missing repos and removes extras.

Local dry run (with the PAT exported):

```bash
export RENOVATE_SYNC_PAT=…   # classic, repo scope
./script/sync-renovate-repos
```

## Adding a repo to the Renovate fleet

1. Land Renovate + aube-lock in that repository (see renovate-config README).
2. Add the repo name under `repos:` in `renovate/repos.yaml`.
3. Merge; sync workflow updates the Mend installation.

## Out of scope

- OpenTofu / Terraform state
- Creating the Mend app itself (marketplace consent only)
- Per-repo aube-lock GitHub Apps (not used — see renovate-config no-App pattern)
