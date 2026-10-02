---
name: actions-router
description: Guide for ComputeSDK Actions — the platform's CI/workflow engine that runs GitHub-style workflow YAML inside sandboxes placed on your org's own compute providers — via the `/api/v1/actions` REST API and the `compute actions` CLI. Use when connecting repos, registering provider credentials, managing vault secrets/variables, dispatching workflow runs, streaming logs, checking flakiness, or fetching artifacts.
---

# Actions Router (Actions API + `compute actions`)

ComputeSDK Actions runs ordinary GitHub Actions workflow YAML (`.github/workflows/*.yml`) inside ComputeSDK sandboxes on the compute providers your organization registers — jobs execute under `act` on real provider VMs and report `provider:region` placement per job. Actions is free for every organization.

Use this skill when driving CI on the ComputeSDK platform: connecting repos, configuring provider credentials, managing the org vault, dispatching runs, streaming logs, judging flakiness, or pulling artifacts. For driving a bare sandbox outside CI, use `sandbox-router`.

## Authentication

- **REST:** `Authorization: Bearer <org API key>` on `https://platform.computesdk.com/api/v1/actions/*`. Owner/admin keys are required for provider credentials and vault writes. Errors: `{ "error": "<message>" }` with 400/401/403/404/410/429/500. Resources in other orgs 404.
- **CLI:** `pnpm dlx @computesdk/cli` or `npm i -g @computesdk/cli` (binary: `compute`). Auth: `--api-key` → `COMPUTE_API_KEY` → `BENCHMARKS_PLATFORM_API_KEY` → stored OAuth (`compute bench auth login`). The gateway key from `compute login` is a different credential and is **not** used. Every command accepts `--json` (failures come back on stderr as `{ "ok": false, "error": {...} }`), `--base-url`, and `--allow-untrusted-host`.

## Repos

```bash
compute actions repos                                  # connected repos
compute actions repos connect <clone-url> [--name] [--branch]
        [--token|--username|--ssh-key-file|--known-hosts-file]
compute actions repos enable|disable <owner>/<repo> [--repo-id]
```

Connect via the platform GitHub App (grants repo access + push/PR triggers) or a generic git remote — http(s) `token`/`basic`, `ssh`, or unauthenticated. `connect` validates with a real `ls-remote` and lands enabled. `POST /actions/repos` does the same over the API; `PATCH` toggles enabled.

## Provider credentials and eligibility

```bash
compute actions providers                              # providers, regions, act status
compute actions providers configure <provider> --key <key> [--verify]
compute actions providers configure <provider> --field name=value ...   # multi-field
compute actions providers verify <provider>            # probe with a real sandbox
compute actions providers remove <provider>            # refused while live boxes need it
```

```
GET          /v1/actions/providers
PUT/DELETE   /v1/actions/providers/{provider}/key
POST         /v1/actions/providers/{provider}/verify
```

Credentials are BYOK: stored encrypted, never readable again. Jobs walk the org's **Actions provider order** (`provider[:region]` entries; `market` bids on the compute market) and every refusal is recorded on the job's `placementAttempts`. Eligibility on top of a stored key: **act-proven** (a real dockerd + act bring-up — Vercel/Tensorlake/Blaxel qualify built-in; other providers need `verify`) and **reconnect-capable** for jobs whose `timeout-minutes` outlives one executor invocation.

## Vault — secrets and variables

```bash
compute actions vault ls [--repo o/r] [--kind secret|variable]
compute actions vault set <name> [--repo] [--kind] [--revealable] [--description] [--labels]
        # value from stdin or --from-file, never an argument; stored exactly as read
compute actions vault get <name> [--repo] [--kind]
compute actions vault rm <name> [--repo] [--kind]
```

`GET/PUT/DELETE /v1/vault?repo=&kind=`. Secrets reach workflows as `${{ secrets.NAME }}` and are masked in logs; write-only unless created `--revealable` (fixed at creation). Variables (`vars.NAME`) are always readable. A repo item overrides the same-named org item. `vault set/get` only ever talk to computesdk.com/loopback hosts — even with `--allow-untrusted-host`.

Workflow secret scoping: the platform scans each workflow for literal `secrets.NAME` references and hands the job exactly those (plus `GITHUB_TOKEN`); unresolvable dynamic access widens to all secrets, or declare it yourself with `# computesdk:secrets=NAME,...` / `=all` / `# computesdk:secrets-env=declared` in the workflow file. A declared-but-unconfigured name resolves to `''` (GitHub parity). Fork PRs get no secrets and no token.

## Dispatch and follow runs

```bash
compute actions dispatch <repo> --workflow <path|name|id> [--ref]
        [--inputs k=v ...] [--manual]
        [--provider <id>[:<region>]] [--max-bid <usd> --max-bid-per second|minute|hour]
compute actions runs <repo> [--status passed|failed|running|cancelled|none ...] [--branch ...]
compute actions history <repo> --workflow <w> [--branch] [--job] [--limit n]
compute actions run <run-id>        # jobs, placement, attempts
compute actions summary <run-id>    # failure digest: failed jobs/steps + redacted log tails
compute actions inspect <run-id>    # resolved image, caches, secret names, concurrency
compute actions logs <run-id> [--job] [--step <n|runner>] [--follow]
compute actions cancel <run-id>
compute actions rerun <run-id>      # same commit, deduped by requestId
compute actions artifacts <run-id> [--job] [--out <dir>]
```

```
POST /v1/actions/dispatch        { workflowId, ref, inputs?, manual?, requestId?,
                                   provider?, providerRegion? } → { runId, created, headSha }
GET  /v1/actions/workflows?repo= workflows on enabled repos; dispatchable:false → manual:true
GET  /v1/actions/runs · /runs/day/{YYYY-MM-DD} · /run-days · /run-states · /history
GET  /v1/actions/runs/{runId} · /state · /summary · /stream (SSE, resumable via ?watch=)
POST /v1/actions/runs/{runId}/cancel · /rerun
GET  /v1/actions/jobs/{jobId}/logs?offset=&step=        byte-addressed CiLogSlice
GET  /v1/actions/jobs/{jobId}/logs?follow=1 · /logs/download
GET  /v1/actions/jobs/{jobId}/artifacts · /artifacts/{artifactId} (signed redirect, 410 expired)
```

- `--workflow` matches file path, display name, or id — not basename. `--manual` runs workflows without `workflow_dispatch` (no `--inputs`); `requestId` dedupes dispatch/rerun.
- `dispatch --provider` pins placement to one provider — a refusal lands on the job's `failureReason`, useful when evaluating providers.
- Logs are byte-addressed: poll `logs?offset=` and resume at `nextOffset`; `--follow`/SSE is sugar over the same offsets. Dashboard URL: `https://platform.computesdk.com/<org>/actions/runs/<runId>`.
- **Before "fixing" a failure, check `history`** — per-job/per-step failure rollups over recent runs tell a regression from a flaky step. `failedBeforeSteps` counts jobs that failed with no step blamed (placement refused, sandbox lost) — platform flakiness, not broken code.

## GitHub compatibility highlights

Supported: `needs:` DAG, `if:`/`failure()`/`always()`/`cancelled()`, `continue-on-error`, matrix (include/exclude), `env`/`GITHUB_ENV`/`GITHUB_PATH`/`GITHUB_OUTPUT`/step outputs/`needs.*.outputs`, `concurrency` + `cancel-in-progress`, `timeout-minutes`, `workflow_dispatch` inputs, third-party `uses:`, `container:` pins, check-run reporting, `actions/checkout` `fetch-depth`. Refused before placement: `services:`, reusable-workflow `uses:`, `environment:`, non-literal `container:`.

## References

- Actions docs: https://github.com/computesdk/computesdk/blob/main/docs/platform/actions.md
- CLI docs: https://github.com/computesdk/computesdk/blob/main/docs/platform/cli.md
- API reference: https://github.com/computesdk/computesdk/blob/main/docs/platform/api-reference.md
