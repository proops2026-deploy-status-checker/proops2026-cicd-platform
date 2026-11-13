# proops2026-cicd-platform

Shared CI/CD for the Deploy Status Checker services: composite actions (step-level reuse) and
reusable workflows (job-level reuse). Service repos call these as thin callers pinned to a tag
(`@v1`), never `@main`.

| Path | Type | Purpose |
|---|---|---|
| `.github/actions/build-app` | Composite action | Node setup + `npm ci` + optional `prepare-script` (e.g. `prisma:generate`) + one npm script (`lint`, `test`, `test:coverage`) |
| `.github/actions/docker-build-push` | Composite action | Buildx + optional login + build/push, tag = commit SHA (default registry Docker Hub) |
| `.github/workflows/reusable-app-ci.yml` | Reusable workflow | `changes` (job-level path routing) → `lint` ∥ `test` ∥ `audit` → `build-image`; `sonar` after `test` |
| `.github/actions/tf-action` | Composite action | Terraform 1.14.9 + AWS creds + `init` (S3 backend) + `fmt -check` + `validate` |
| `.github/workflows/reusable-iac.yml` | Reusable workflow | `plan` → apply the saved plan (`apply`), or `plan -destroy` → apply (`destroy`) |

## Versioning

- Immutable release tags: `v1.0.0` style (e.g. `v1.1.0`). Never moved.
- Floating major tag `v1` = latest `v1.x.y`. Moved only for backward-compatible changes.
- Breaking change → new major (`v2`); services opt in by editing `@v1` → `@v2`.
- Composite actions inside the reusable workflows are referenced as `.../.github/actions/<name>@v1`
  (absolute path): inside a reusable workflow `./` resolves to the caller's checkout.
