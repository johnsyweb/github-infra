# johnsyweb/github-infra

Declarative GitHub account wiring for johnsyweb. Today this manages **which repositories** the [Mend Renovate](https://github.com/apps/renovate) app can access.

Shared Renovate *config* and the no-App aube-lock workflow live in [johnsyweb/renovate-config](https://github.com/johnsyweb/renovate-config).

## One-time: install Mend Renovate

1. Open [github.com/apps/renovate](https://github.com/apps/renovate) → **Configure** / **Install**.
2. Choose **Only select repositories**.
3. Select **ambassy** (or leave empty if the API will add it — GitHub may require at least one; pick ambassy).
4. Save.

### Find `installation_id`

`GET /user/installations` only accepts a **GitHub App user-to-server** token, so a normal `gh auth` / PAT call returns HTTP 403. Read the id from the browser instead:

1. Open [Installed GitHub Apps](https://github.com/settings/installations) (or [Configure Renovate](https://github.com/apps/renovate)).
2. Click **Configure** next to Renovate.
3. The URL is `https://github.com/settings/installations/<installation_id>` — use that number.

Put it into [`renovate/repos.yaml`](renovate/repos.yaml) as `installation_id`.

### Create `RENOVATE_SYNC_PAT`

GitHub’s installation membership endpoints require a **classic** PAT (not fine-grained, not `GITHUB_TOKEN`) with:

| Scope | Why |
| --- | --- |
| `repo` | Add/remove repositories on the installation |
| `read:user` | List repositories already on the installation |

Create at [github.com/settings/tokens](https://github.com/settings/tokens) (classic), then:

```bash
gh secret set RENOVATE_SYNC_PAT --repo johnsyweb/github-infra
```

Paste the token when prompted. Rotate periodically. If sync fails with HTTP 403 mentioning `read:user`, recreate the PAT with both scopes and set the secret again.

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
export RENOVATE_SYNC_PAT=…   # classic: repo + read:user
./script/sync-renovate-repos
```

## Adding a repo to the Renovate fleet

1. Land Renovate + aube-lock in that repository, including the README refresh gate (see [renovate-config README — Use in a repo](https://github.com/johnsyweb/renovate-config#use-in-a-repo)).
2. Add the repo name under `repos:` in `renovate/repos.yaml`.
3. Merge; sync workflow updates the Mend installation.

## Out of scope

- OpenTofu / Terraform state
- Creating the Mend app itself (marketplace consent only)
- Per-repo aube-lock GitHub Apps (not used — see renovate-config no-App pattern)
