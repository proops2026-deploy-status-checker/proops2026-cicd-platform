# proops2026-cicd-platform

Shared CI/CD for the Deploy Status Checker services: composite actions (step-level reuse) and
reusable workflows (job-level reuse). Service repos call these as thin callers pinned to a tag
(`@v2`; `@v1` still works for v1 callers), never `@main`.

| Path | Type | Purpose |
|---|---|---|
| `.github/actions/build-app` | Composite action | Node setup + `npm ci` + optional `prepare-script` (e.g. `prisma:generate`) + one npm script (`lint`, `test`, `test:coverage`) |
| `.github/actions/docker-build-push` | Composite action | Buildx + optional login + build/push, tag = commit SHA (default registry Docker Hub) |
| `.github/workflows/reusable-build.yml` | Reusable workflow | (v1: `reusable-app-ci.yml`) `changes` (job-level path routing) → `lint` ∥ `test` ∥ `audit` → `build-image`; `sonar` after `test`. **Outputs** `image-tag` (full SHA) + `image-digest`, empty unless pushed. Optional `postgres-image` + `migrate-script`: throwaway PostgreSQL service + migration for integration tests |
| `.github/workflows/reusable-deploy.yml` | Reusable workflow | Promote a pushed `<image>:<sha>` to `dev` / `staging` / `prod` (env = GitHub Environment = K8s namespace). Never builds. Callers pass `secrets: inherit`. Kubeconfig in `$RUNNER_TEMP`, app secrets → K8s Secret, image pinned + `change-cause`, rollout + tag/digest verify. Default runner `[self-hosted, minikube-deploy]` |
| `.github/actions/tf-action` | Composite action | Terraform 1.14.9 + AWS creds + `init` (S3 backend) + `fmt -check` + `validate` |
| `.github/workflows/reusable-iac.yml` | Reusable workflow | `plan` → apply the saved plan (`apply`), or `plan -destroy` → apply (`destroy`) |

## Versioning

- Immutable release tags: `v1.0.0` style (e.g. `v1.1.0`). Never moved.
- Floating major tag `v1` = latest `v1.x.y`. Moved only for backward-compatible changes.
- Breaking change → new major (`v2`); services opt in by editing `@v1` → `@v2`.
- `v2.0.0` (breaking): `reusable-app-ci.yml` renamed to `reusable-build.yml` (+ outputs), `reusable-deploy.yml` added.
  `v1` stays at `v1.4.0` for callers not yet migrated.
- Composite actions inside the reusable workflows are referenced as `.../.github/actions/<name>@v2`
  (absolute path): inside a reusable workflow `./` resolves to the caller's checkout.
