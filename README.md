# go-zoen-ci

**Public dual-path CI** for private [`EnzoTironi/go-zoen`](https://github.com/EnzoTironi/go-zoen).

Standard GitHub-hosted runners on **public** repos do not consume paid Actions minutes on the private account. This repo only holds workflows; application code is checked out from the private repo via PAT.

Same pattern as [`buzz-bok-ci`](https://github.com/EnzoTironi/buzz-bok-ci) for BOK.

## Flow

```
private go-zoen (source)  ──PAT checkout──►  this public repo’s Actions
                                                    │
                                                    └── quality matrix
                                                        (prek, lint, unit, race,
                                                         govulncheck, gitleaks, actionlint)
                                                    └── commit status → go-zoen@SHA
                                                        context: zoen-ci/public
```

## Secret (required)

| Secret | Purpose |
|--------|---------|
| `PRIVATE_REPO_TOKEN` | Fine-grained PAT: **Contents: Read** on `go-zoen`, **Commit statuses: Read and write** (required) |

Create at: https://github.com/settings/personal-access-tokens

Then: [Settings → Secrets → Actions](https://github.com/EnzoTironi/go-zoen-ci/settings/secrets/actions) → `PRIVATE_REPO_TOKEN`

## Trigger from a go-zoen checkout

```bash
# current HEAD
./scripts/zoen-trigger-public-ci.sh

# explicit ref / SHA
./scripts/zoen-trigger-public-ci.sh --ref main
./scripts/zoen-trigger-public-ci.sh --ref feat/compiler-c8-cli-hf-bundle

make ci-remote

gh run list --repo EnzoTironi/go-zoen-ci --limit 8
```

## Local (always free)

```bash
make check
make check-full
```

## Notes

- Private-repo `quality.yml` is **manual only** (`workflow_dispatch`) so merges never burn paid private minutes.
- Set private account Actions spending limit to $0 if you want a hard backstop.
- Docs in private repo: `docs/operations/public-ci-dual-path.md`

## Confidentiality limitation

This repository and its Actions logs are public. The workflow does not persist the private checkout credential and does not publish source artifacts, but command output, test names, file paths, diagnostics, and failure messages can become public. Dispatch only private-source commits whose CI output is suitable for public disclosure. Never print source contents or secrets from tests and gates; diagnose sensitive failures locally.

This is residual risk in the current public-runner design. SHA pinning and `persist-credentials: false` protect integrity and credentials, not the confidentiality of process output.

## Trust boundary

- The source repository is hardcoded to `EnzoTironi/go-zoen`.
- Runs accept only a full lowercase 40-character commit SHA.
- Every checkout uses `persist-credentials: false`.
- The workflow verifies the resolved checkout SHA before testing or reporting status.
- Failure to publish `zoen-ci/public` fails the workflow; a green run therefore proves the status was written.
- Superseded runs are cancelled and do not overwrite the newer run's status.
