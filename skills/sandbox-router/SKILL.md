---
name: sandbox-router
description: Guide for the ComputeSDK Platform Sandboxes API and `compute sandboxes` CLI — place a sandbox on your org's own provider credentials (or the compute market) over `/api/v1/sandboxes`, then exec commands, manage detached processes with interactive stdin, drive the filesystem, resolve port URLs, and take snapshots. Use when creating, driving, or pricing platform-routed sandboxes rather than talking to a provider SDK directly.
---

# Sandbox Router (Sandboxes API + `compute sandboxes`)

The ComputeSDK Platform's sandbox router places a sandbox for your organization the same way an Actions job gets one — walking your org's provider order over your own provider credentials (BYOK), falling back to market fills, or claiming a pre-warmed box from your pool — and then hands you the box: exec commands, spawn detached processes, read/write files, resolve port URLs, snapshot it, and see what it cost.

Use this instead of the `computesdk` skill's per-provider SDK path when you want the platform to pick and place the provider, audit every command, and settle cost — or when you want one API that works identically across every provider.

## Authentication

Two surfaces, same auth:

- **REST:** `Authorization: Bearer <org API key>` against `https://platform.computesdk.com/api/v1`. Errors are `{ "error": "<message>" }` with 400/401/403/404/409/410/429/500.
- **CLI:** `pnpm dlx @computesdk/cli` or `npm i -g @computesdk/cli` (binary: `compute`). Auth resolves `--api-key` → `COMPUTE_API_KEY` → `BENCHMARKS_PLATFORM_API_KEY` → stored OAuth from `compute bench auth login`. Every subcommand takes `--json`, `--base-url`, and `--allow-untrusted-host` (explicit keys only; stored login is never sent to non-computesdk hosts).

Creating a sandbox and running commands require the org's **`sandboxes` feature flag**; list/get/costs/destroy stay open so disabling the flag never strands a running box. Sandboxes run on the org's own provider keys (manage them under Settings → Compute or `compute actions providers`), except ambient providers and market fills.

## Provider order and market

Each create walks a provider order — `provider` or `provider:region` entries, e.g. `market,blaxel,vercel` — recording every refusal in `placementAttempts`. The special entry `market` bids on the compute market's open asks before continuing down the order. Resolution order: per-request `providerOrder` → the org's sandbox order (`sandbox_settings`, set via `PATCH /api/v1/sandboxes/settings`) → the Actions provider order → the deployment default (`["vercel"]`).

```bash
compute sandboxes create --order market,namespace,vercel --label devbox
```

## Core lifecycle

```bash
compute sandboxes create [--order <providers>] [--label <l>] [--image <img>]
                         [--snapshot-id <id>] [--cpus <n>] [--memory-mb <n>]
                         [--disk-mb <n>] [--timeout-ms <ms>] [--secret <name>...]
compute sandboxes list [--status creating|running|destroyed] [--limit n] [--cursor c]
compute sandboxes get <sandboxId>        # includes the BYOK `attach` descriptor
compute sandboxes destroy <sandboxId>    # verified teardown + cost settle
```

```
POST   /api/v1/sandboxes        { label?, providerOrder?, image?, snapshotId?,
                              resources?: {cpus,memoryMb,ephemeralDiskMb},
                              timeoutMs?, secrets?: [vaultNames] }
GET    /api/v1/sandboxes?status=&limit=&cursor=
GET    /api/v1/sandboxes/{id}   → sandbox + attach: {provider, providerSandboxId, region} | null
DELETE /api/v1/sandboxes/{id}   → settles cost
GET    /api/v1/sandboxes/costs  → { running, settled, totalUsd, byProvider }
```

- `attach` lets a BYOK customer drive the box directly via the provider SDK (`connect()` with `providerSandboxId` + `region`); it is `null` on market fills (the seller's credential is never handed out), ambient providers, and non-running boxes. Direct attach bypasses command audit and vault secret injection.
- `secrets` names org-vault secrets to inject as env vars (see the `actions-router` skill for the vault CLI).
- `timeoutMs` caps a forgotten sandbox at the provider (default 30 min, max 6 h).
- Every sandbox carries `cost: { rate, runtimeSeconds, costUsd, settled }` — an org-set or platform-default per-provider rate snapshotted at placement, applied to wall-clock lifetime.
- `requestedImage` vs `image`: what you asked for vs what the provider actually booted (Blaxel/Tensorlake ignore `image`; Vercel reports `null`).

## Commands and processes

```bash
compute sandboxes exec <id> <command...> [--timeout-ms]      # one-shot, prints result
compute sandboxes spawn <id> <command...> [--cwd] [-e K=V]... [--stdin]
compute sandboxes ps <id>                                   # tracked processes
compute sandboxes logs <id> <jobId> [--follow]              # buffered output / stream
compute sandboxes wait <id> <jobId> [--timeout-ms]
compute sandboxes kill <id> <jobId> [--signal SIGKILL]
compute sandboxes stdin <id> <jobId> [--data|--file|--base64|piped]
compute sandboxes close-stdin <id> <jobId>
```

```
POST /api/v1/sandboxes/{id}/commands  { command, timeoutMs? } → SSE (stdout/stderr/exit/error)
                                                            or JSON with Accept: application/json
POST /api/v1/sandboxes/{id}/processes           { command, cwd?, env?, stdin? } → 201 { jobId }
GET  /api/v1/sandboxes/{id}/processes           list tracked jobs
GET  /api/v1/sandboxes/{id}/processes/{jobId}   status snapshot; Accept: text/event-stream to follow
POST /api/v1/sandboxes/{id}/processes/{jobId}/wait         { timeoutMs? }
POST /api/v1/sandboxes/{id}/processes/{jobId}/kill         { signal? } (default SIGTERM)
POST /api/v1/sandboxes/{id}/processes/{jobId}/stdin        { data, encoding?: "utf8"|"base64" } (1 MiB max)
POST /api/v1/sandboxes/{id}/processes/{jobId}/close-stdin
```

- Processes run detached through the in-sandbox daemon and outlive the request; exited jobs expire daemon-side after ~10 min — after that the audit row is the record.
- `stdin: true` keeps the job's stdin pipe open for `stdin`/`close-stdin` (a process spawned without it 409s). Binary-safe via `--base64` / `encoding: "base64"`.
- A dropped SSE connection does not abort the command — it runs to completion or `timeoutMs`.

## Filesystem and URLs

```bash
compute sandboxes ls <id> [path]
compute sandboxes cat <id> <path>
compute sandboxes write <id> <path> [--content|--file|stdin]
compute sandboxes mkdir <id> <path>
compute sandboxes rm <id> <path>
compute sandboxes url <id> --port <n> [--protocol <p>]
```

```
GET    /api/v1/sandboxes/{id}/files?path=  → {type:"file",content} | {type:"directory",entries}
POST   /api/v1/sandboxes/{id}/files        { path, content } | { path, mkdir: true }
DELETE /api/v1/sandboxes/{id}/files?path=
GET    /api/v1/sandboxes/{id}/urls?port=&protocol=  → { url }
```

Providers without a filesystem or public URL surface answer 501.

## Snapshots (BYOK only)

```bash
compute sandboxes snapshots <id>                      # list
compute sandboxes snapshot <id> [--label <l>]         # snapshot a running box
compute sandboxes snapshot-delete <id> <snapshotId>
compute sandboxes create --snapshot-id <providerSnapshotId>   # resume from one
```

```
POST   /api/v1/sandboxes/{id}/snapshots                    { label? }
GET    /api/v1/sandboxes/{id}/snapshots
DELETE /api/v1/sandboxes/{id}/snapshots/{snapshotRowId}
```

BYOK only — a market-filled box answers 409 (the artifact would land in the seller's account). Snapshot-capable providers: archil, blaxel, namespace, tensorlake, vercel; others answer 501. A provider whose snapshot call stops the source box (vercel) settles the sandbox row so it stops billing.

## Settings, rates, warm pool

```
GET/PATCH /api/v1/sandboxes/settings   { providerOrder?, marketCap?, providerResources?, warmPool? }
GET/PUT/DELETE /api/v1/sandboxes/rates { provider, rate, per: second|minute|hour }  (org overrides)
GET/POST  /api/v1/sandboxes/pool/fill  report floors + inventory; trigger an ensure now
```

- `providerOrder` — the org's sandbox order; empty list inherits the Actions order.
- `marketCap` — max $/vCPU-time for `market` order entries (required when the order uses `market`).
- `providerResources` — per-provider default sizes.
- `warmPool` — `provider[:region] → count` floor of pre-warmed boxes a plain create claims instead of cold-placing. Creates carrying `image`/`snapshotId`/`resources`/`secrets` skip the pool; labels under `sb-pool` are reserved.

## Failure semantics worth knowing

- Placement exhausted → `502` with `details.attempts` (each provider's refusal reason).
- Destroy verifies the provider no longer knows the box; a surviving box keeps the row `running` and answers `502` to retry — a row only leaves `running` once the provider confirms it's gone, which is what stops billing.
- Commands against a box the provider can't look up → `503`; a stopped-but-listed box → `409`.
- A provider key can't be removed while the org has live sandboxes on it (`409` + `details.sandboxIds`).

## References

- CLI docs: https://github.com/computesdk/computesdk/blob/main/docs/platform/cli.md
- API reference: https://github.com/computesdk/computesdk/blob/main/docs/platform/api-reference.md
- Router design: `docs/2026-09-11-sandbox-router.md` in computesdk/benchmarks-platform
