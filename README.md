# proops2026-cicd-platform

Shared CI/CD for the Deploy Status Checker services: composite actions (step-level reuse) and
reusable workflows (job-level reuse). Service repos call these as thin callers pinned to a tag
(`@v1`), never `@main`.

| Path | Type | Purpose |
|---|---|---|
| `.github/actions/build-app` | Composite action | Node setup + `npm ci` + one npm script (`lint`, `test`, `test:coverage`) |
| `.github/actions/docker-build-push` | Composite action | Buildx + optional login + build/push, tag = commit SHA (default registry Docker Hub) |
| `.github/workflows/reusable-app-ci.yml` | Reusable workflow | `lint` ∥ `test` ∥ `audit` → `build-image`; `sonar` after `test` |
