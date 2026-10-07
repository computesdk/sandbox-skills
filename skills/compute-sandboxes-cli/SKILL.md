---
name: compute-sandboxes-cli
description: Drive hosted ComputeSDK Platform sandboxes from a shell with `compute sandboxes` (alias `compute sbx`) from @computesdk/cli 2.x — create sandboxes on your org's providers or the compute market, run commands, manage detached processes and interactive stdin, move files, get preview URLs, and take snapshots. Use when creating, running commands in, or managing hosted ComputeSDK sandboxes from a shell — no per-provider SDK or provider keys needed in your code.
---

# `compute sandboxes` — hosted sandboxes from the shell

The `compute` CLI (`@computesdk/cli`) drives the ComputeSDK Platform at `https://platform.computesdk.com`: it places a sandbox on your organization's own provider credentials (or a compute-market fill) via `/api/v1/sandboxes` and lets you exec commands, spawn processes, move files, resolve preview URLs, and snapshot — all without embedding a provider SDK. Requires @computesdk/cli **2.x**; creating sandboxes requires the org to have **Sandboxes enabled** (403 otherwise), and usage bills to the org.

`compute market` (the sell side of the compute market) exists as a sibling command group but is out of scope for this skill.

## Install, auth, output

```bash
npm i -g @computesdk/cli        # or: npx @computesdk/cli <cmd>
compute --version               # must show 2.x
```

If `--version` shows 1.0.x, an old standalone binary (`~/.local/bin/compute`) is shadowing the npm install — remove it.

- **Interactive:** `compute login` — OAuth device flow (open the URL, enter the code). `compute logout` clears it. One login covers `compute sandboxes`, `compute actions`, `compute market`, and `compute bench`.
- **Non-interactive (CI, agents):** prefer an org API key (create under Settings → API keys on the platform). Credential resolution order: `--api-key <key>` → `COMPUTE_API_KEY` → `BENCHMARKS_PLATFORM_API_KEY` (legacy) → stored `compute login` session.
- **Another platform host:** `--base-url <url>` or `COMPUTE_PLATFORM_URL`. Non-`computesdk.com` hosts also need `--allow-untrusted-host`, which applies to explicit keys only — stored logins are never sent there.
- **Multi-org accounts (2.1+):** `compute org list` shows your orgs, `compute org use <slug>` sets the persisted active org, `compute org current` (or `compute whoami`) shows the user + active org. `--org <slug>` or `COMPUTE_ORG` overrides for a single command; the `compute login` approval screen also offers an org picker.
- **Agents:** always pass `--json` — machine-readable success output, and failures come back on stderr as a single error envelope.
- `insufficient_scope` or 401 errors → run `compute login` again.
- Never echo, print, or commit API keys or secrets; read them from environment variables.

## Commands

```bash
compute sandboxes create [--order market,blaxel,vercel] [--label <l>]
    [--image <img>] [--snapshot-id <id>] [--cpus <n>] [--memory-mb <n>]
    [--disk-mb <n>] [--timeout-ms <ms>] [--secret <vaultName>...]
    # prints the sandbox ID
compute sandboxes list [--status creating|running|destroyed] [--limit n] [--cursor c]
compute sandboxes get <id>          # provider, placement, cost
compute sandboxes destroy <id>      # ALWAYS destroy when done — sandboxes bill while running

compute sandboxes exec <id> <command...> [--timeout-ms]     # one-off, buffered (~290s max)
compute sandboxes spawn <id> <command...> [--cwd] [-e K=V]... [--stdin]  # detached; returns a job ID
compute sandboxes ps <id>
compute sandboxes logs <id> <jobId> [-f|--follow]           # stream until exit
compute sandboxes wait <id> <jobId> [--timeout-ms]          # job keeps running on timeout
compute sandboxes kill <id> <jobId> [--signal SIGKILL]
compute sandboxes stdin <id> <jobId> [--data|--file|--base64|piped]  # needs spawn --stdin
compute sandboxes close-stdin <id> <jobId>

compute sandboxes ls <id> [path]
compute sandboxes cat <id> <path>
compute sandboxes write <id> <path> [--content|--file <local>|stdin]   # absolute paths
compute sandboxes mkdir <id> <path>
compute sandboxes rm <id> <path>

compute sandboxes url <id> --port <n> [--protocol <p>]      # public preview URL
compute sandboxes snapshots <id>                            # list
compute sandboxes snapshot <id> [--label <l>]               # snapshot a running box
compute sandboxes snapshot-delete <id> <snapshotId>
```

## Typical workflow

```bash
ID=$(compute sandboxes create --order market,blaxel,vercel --json | jq -r .sandbox.id)
compute sandboxes write $ID /app/server.py --file ./server.py
compute sandboxes exec $ID pip install flask
compute sandboxes spawn $ID python /app/server.py --cwd /app        # returns a job ID
compute sandboxes logs $ID <jobId> --follow                         # Ctrl-C detaches; the job keeps running
compute sandboxes url $ID --port 5000                               # → https://...
compute sandboxes destroy $ID
```

`logs --follow` blocks until the job exits, so for a long-running server it never returns — detach with Ctrl-C (or run `url`/`destroy` from another shell) before continuing.

create → write/exec → `spawn` for servers → `url --port` → **destroy**.

## Pitfalls

- **~290s ceiling:** `exec` buffers output and tops out around 290 seconds. Anything longer — servers, builds, watchers — goes through `spawn` + `logs -f` or `wait`.
- **Preview URLs:** the server must listen on `0.0.0.0` on the exact `--port`, or the preview 502s. Flask: `app.run(host="0.0.0.0", port=N)`. Vite: `vite --host 0.0.0.0 --port N --strictPort` plus `server.allowedHosts` covering the preview host (e.g. `.bl.run`). Next.js: `next dev -H 0.0.0.0 -p N`.
- **Tensorlake:** `url` makes every exposed port on that sandbox public, not just the one you asked for.
- **`--secret`** names an org-vault secret injected into the sandbox environment (repeatable); manage vault items with `compute actions vault` (see the `compute-actions-cli` skill).
- **Provider order** walks entries 1,2,3… — `market` bids on the compute market first; each refusal is recorded in `placementAttempts`. Snapshots are BYOK-only (market-filled boxes refuse them).
- Destroy is verified — it settles billing only once the provider confirms the box is gone; a 502 means retry.

## REST equivalent

For scripts that skip the CLI: `POST https://platform.computesdk.com/api/v1/sandboxes` with `Authorization: Bearer $COMPUTE_API_KEY` returns `{ sandbox: { id, … } }`. File writes are `POST /api/v1/sandboxes/{id}/files` with `{ path, content }`.

```
POST   /api/v1/sandboxes        { label?, providerOrder?, image?, snapshotId?,
                                resources?: {cpus,memoryMb,ephemeralDiskMb},
                                timeoutMs?, secrets?: [vaultNames] }
GET    /api/v1/sandboxes?status=&limit=&cursor=
GET    /api/v1/sandboxes/{id}   → sandbox + attach
DELETE /api/v1/sandboxes/{id}   → settles cost
GET    /api/v1/sandboxes/costs  → { running, settled, totalUsd, byProvider }
```

### Provider order

Each create walks a provider order — `provider` or `provider:region` entries — recording every refusal in `placementAttempts`. The special entry `market` bids on the compute market's open asks before the walk continues. Resolution order: per-request `providerOrder` (the CLI's `--order`) → the org's own sandbox order → the Actions provider order → the deployment default (`["vercel"]`).

### Settings, rates, warm pool

```
GET/PATCH /api/v1/sandboxes/settings   { providerOrder?, marketCap?, providerResources?, warmPool? }
GET/PUT/DELETE /api/v1/sandboxes/rates { provider, rate, per: second|minute|hour }  (org overrides)
GET/POST  /api/v1/sandboxes/pool/fill  report floors + inventory; trigger an ensure now
```

- `providerOrder` — the org's sandbox order; an empty list inherits the Actions order.
- `marketCap` — max $/vCPU-time for `market` order entries (required when the order uses `market`).
- `providerResources` — per-provider default sizes.
- `warmPool` — `provider[:region] → count` floor of pre-warmed boxes a plain create claims instead of cold-placing. Creates carrying `image`/`snapshotId`/`resources`/`secrets` skip the pool; labels under `sb-pool` are reserved.
- Cost model: a per-provider rate (org override → platform default) is snapshotted onto the sandbox at placement and applied to wall-clock lifetime; every sandbox carries `cost: { rate, runtimeSeconds, costUsd, settled }`.

### The `attach` descriptor

`POST`/`GET` return `sandbox.attach: { provider, providerSandboxId, region } | null` — with your own provider key (BYOK) you can drive the box directly via the provider's SDK (`connect()` with the id + region) instead of proxying through the platform. `attach` is `null` on market fills (the seller's credential is never handed out), ambient providers, and non-running boxes. Direct attach bypasses command audit and vault `secrets` env injection.
