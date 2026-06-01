# Release And CI

Use this reference for Rute Bayar PR, CI, release, and package distribution workflows.

## CI

Expected baseline:

```bash
go test ./...
./scripts/smoke-local.sh
npm --prefix site run build
```

GitHub Actions:

```text
.github/workflows/ci.yml
.github/workflows/e2e-sandbox.yml
.github/workflows/release.yml
.github/workflows/site-pages.yml
```

`CI` runs unit/build checks and local webhook forwarding smoke. `E2E Sandbox` is separate because it requires external provider credentials.

## E2E Sandbox

Sandbox E2E can run automatically for trusted internal PRs or manually with inputs:

```bash
gh workflow run "E2E Sandbox" \
  -f run_xendit=true \
  -f run_midtrans=false
```

Credentials should live in GitHub Secrets or local environment variables, never in repo files.

References must be unique per run. Prefer UTC timestamp plus GitHub run id/attempt to avoid duplicate provider reference collisions.

## PR Flow

Recommended flow:

```bash
git switch main
git pull --ff-only
git switch -c feat/<task-name>
# edit, test
git add <files>
git commit -m "<scope>: <summary>"
git push -u origin feat/<task-name>
gh pr create --base main --head feat/<task-name> --title "<title>" --body-file <body-file>
```

Before merge:

```bash
gh pr checks <number> --repo pendig/rute-bayar
gh pr view <number> --repo pendig/rute-bayar --json mergeStateStatus,reviewDecision,statusCheckRollup
```

Inspect review threads when comments exist; fix actionable comments, commit, push, and reply.

## Release Flow

Prepare release docs first:

- `CHANGELOG.md`
- `README.md` version/status links
- `site/src/data/changelog.ts`

Merge release PR to main, then tag:

```bash
git switch main
git pull --ff-only
git tag -a vX.Y.Z -m "vX.Y.Z"
git push origin vX.Y.Z
```

The Release workflow should:

- run formatting/vet/tests
- build release binaries
- publish GitHub Release
- sync Homebrew tap when `HOMEBREW_TAP_TOKEN` is configured

Verify:

```bash
gh run list --workflow Release --limit 5
gh release view vX.Y.Z --json tagName,name,url,isPrerelease,assets
gh pr list --repo pendig/homebrew-tap --state open
```

If a Homebrew tap PR is opened, verify the formula diff only bumps version and checksums, then merge it.

## Release Readiness

Do not call a release ready only because CI is green. Also check:

- provider support notes are truthful
- known limitations are documented
- sandbox proof is described accurately
- no secrets are exposed
- open PRs that affect release are merged or deliberately postponed
- Homebrew formula points to the new version when distribution was promised
