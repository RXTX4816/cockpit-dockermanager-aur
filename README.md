# cockpit-dockermanager-aur

Maintains the [`cockpit-dockermanager`](https://aur.archlinux.org/packages/cockpit-dockermanager) AUR
package, tracking releases of [chrisjbawden/cockpit-dockermanager](https://github.com/chrisjbawden/cockpit-dockermanager).

## How it works

Two workflows, two-step human-gated flow:

1. **Check upstream release** (`.github/workflows/check-release.yml`) — runs every 6 hours, and can
   be run manually. It compares the latest stable upstream release tag against `pkgver` in
   [PKGBUILD](PKGBUILD). If there's a new version, it opens a GitHub Issue (label `release-update`)
   with the release notes, a compare link, and the last 5 releases for context. This step never
   modifies any files or builds/pushes anything.

2. **Publish to AUR** (`.github/workflows/publish-aur.yml`) — manual only (`workflow_dispatch`),
   takes the release tag as input (e.g. `v1.0.8.2`). The job runs under the `aur-publish`
   GitHub Environment, which requires a reviewer to approve the run before it starts. Once
   approved, it builds the package in an `archlinux:base-devel` container, generates `.SRCINFO`,
   pushes both files to AUR over SSH, commits the updated `PKGBUILD`/`.SRCINFO` back to `main`,
   and closes the tracking issue.

**Nothing reaches AUR without an explicit approval click in the Actions tab.**

## One-time repo setup

1. **Environment + reviewer** — Settings → Environments → New environment → name it
   `aur-publish` → under "Deployment protection rules" add yourself as a required reviewer.
2. **AUR SSH key** — Settings → Secrets and variables → Actions. Add a secret named
   `AUR_SSH_KEY` containing the private key registered against your AUR account (the same key
   used by the Forgejo pipeline). If your plan supports environment-scoped secrets, scope it to
   `aur-publish` so it's only ever readable by the gated job.

## Publishing a release

1. Wait for (or manually trigger) **Check upstream release** to open an issue for the new
   version.
2. Go to **Actions → Publish to AUR → Run workflow**, enter the version tag from the issue
   (e.g. `v1.0.8.3`).
3. Approve the `aur-publish` environment review when prompted — this is where you decide
   whether the release actually goes out.
4. Watch the run; it builds, pushes to AUR, syncs `PKGBUILD`/`.SRCINFO` back to `main`, and
   closes the tracking issue automatically.
