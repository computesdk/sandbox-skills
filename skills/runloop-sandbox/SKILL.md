---
name: runloop-sandbox
description: Guide for creating and managing Runloop Devbox sandboxes using ComputeSDK. Use when building applications that need Runloop's secure, isolated sandbox environments (Devboxes) for AI agent workflows, code execution, and development environments with features like blueprints, snapshots, tunnels, and named shells.
---

# Runloop Sandboxes with ComputeSDK

Run code in Runloop's secure, isolated Devbox environments through ComputeSDK's unified API. Runloop provides lightning-fast sandboxed execution environments designed for AI agent workflows — ideal for running AI-generated code, executing tests, building development environments, and powering coding agents with features like blueprints, snapshots, suspend/resume, tunnels, and named shells.

## Setup

```bash
npm install computesdk
```

```bash
# .env
COMPUTESDK_API_KEY=your_computesdk_api_key
RUNLOOP_API_KEY=your_runloop_api_key
```

Get your ComputeSDK key at https://console.computesdk.com/register
Get your Runloop API key at https://app.runloop.ai (Settings page on the Runloop Dashboard)

## Quick Start

```typescript
import { compute } from 'computesdk';
// Auto-detects Runloop from environment variables

const sandbox = await compute.sandbox.create();

const result = await sandbox.runCode('print("Hello from Runloop!")');
console.log(result.output);

await sandbox.destroy();
```

## Explicit Configuration

For multi-provider setups or when you want to be explicit:

```typescript
import { compute } from 'computesdk';

compute.setConfig({
  computesdkApiKey: process.env.COMPUTESDK_API_KEY,
  provider: 'runloop',
  runloop: {
    apiKey: process.env.RUNLOOP_API_KEY,
  }
});

const sandbox = await compute.sandbox.create();
```

## Runloop Configuration Options

```typescript
interface RunloopConfig {
  apiKey?: string;              // Uses RUNLOOP_API_KEY env var if not set
  runtime?: 'node' | 'python'; // Auto-detects from code patterns
  timeout?: number;             // Execution timeout in ms
}
```

## Runloop-Specific Features (Native SDK)

For Runloop-specific features not available through ComputeSDK's unified API, use the Runloop TypeScript SDK directly:

```bash
npm install @runloop/api-client
```

### Create a Devbox and Execute Commands

```typescript
import { RunloopSDK } from '@runloop/api-client';

const runloop = new RunloopSDK();

// Create a devbox and wait for it to be ready
const devbox = await runloop.devbox.create();

// Execute a command
const result = await devbox.cmd.exec('echo "Hello from Runloop!"');
console.log(await result.stdout()); // "Hello from Runloop!"
console.log(result.exitCode);       // 0

// Run async commands (long-running processes)
const serverExec = await devbox.cmd.execAsync('npx http-server -p 8080');
console.log(`Started server: ${serverExec.executionId}`);

// Kill async command when done
await serverExec.kill();

await devbox.shutdown();
```

### Named Shells (Stateful Sessions)

Named shells maintain environment variables and working directory across commands:

```typescript
const shell = devbox.shell('my-session');

await shell.exec('cd ~/my-project');
await shell.exec('export NODE_ENV=production');

// State is preserved
const pwdResult = await shell.exec('pwd');
console.log(await pwdResult.stdout()); // /home/user/my-project

const envResult = await shell.exec('echo $NODE_ENV');
console.log(await envResult.stdout()); // production
```

### File Operations

```typescript
// Write a file
await devbox.file.write({
  filePath: '/home/user/main.py',
  contents: 'print("Hello, World!")',
});

// Read a file
const contents = await devbox.file.read({
  filePath: '/home/user/test_results.txt',
});
```

### Blueprints (Custom Environment Templates)

```typescript
const blueprint = await runloop.blueprint.create({
  name: 'my-blueprint',
  dockerfile: 'FROM ubuntu:22.04\nRUN apt-get update',
});

// Create a devbox from the blueprint
const devbox = await blueprint.createDevbox({ name: 'my-devbox' });
```

### Snapshots (Save and Restore Disk State)

```typescript
// Create a snapshot of the current devbox state
const snapshot = await devbox.snapshotDisk();

// Create a new devbox from the snapshot
const newDevbox = await snapshot.createDevbox();
const contents = await newDevbox.file.read('hello.txt');
```

### Tunnels (Expose Services)

```typescript
// Enable tunnel at devbox creation
const devbox = await runloop.devbox.create({
  tunnel: { authMode: 'open' },
  entrypoint: 'python3 -m http.server 8080 --bind 0.0.0.0',
});

// Access tunnel URL: https://8080-{tunnel_key}.tunnel.runloop.ai
const info = await devbox.getInfo();
const tunnelUrl = `https://8080-${info.tunnel.tunnelKey}.tunnel.runloop.ai`;
```

### Instance Sizes

```typescript
const devbox = await runloop.devbox.create({
  launchParameters: {
    resourceSizeRequest: 'MEDIUM', // X_SMALL | SMALL | MEDIUM | LARGE | X_LARGE | XX_LARGE
  },
});
```

| Size | CPU | Memory | Storage |
|------|-----|--------|---------|
| X_SMALL | 0.5 | 1GB | 4GB |
| SMALL | 1 | 2GB | 4GB |
| MEDIUM | 2 | 4GB | 8GB |
| LARGE | 2 | 8GB | 16GB |
| X_LARGE | 4 | 16GB | 16GB |
| XX_LARGE | 8 | 32GB | 16GB |

### Suspend and Resume

```typescript
// Suspend a running devbox (preserves disk state)
await devbox.suspend();

// Resume later
await runloop.api.devboxes.resume(devboxId);
```

## Full API

ComputeSDK provides the same API across all providers: filesystem operations, shell commands, managed servers, overlays, terminals, and client access.

Install the main skill for the complete reference:

```
npx skills add https://github.com/computesdk/sandbox-skills --skill computesdk
```

Or see https://www.computesdk.com/docs/reference/sandbox/

## Runloop Documentation

- Runloop Docs: https://docs.runloop.ai
- TypeScript SDK: https://github.com/runloopai/api-client-ts
- Dashboard: https://app.runloop.ai
