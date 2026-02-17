---
name: modal-sandbox
description: Guide for creating and managing Modal sandboxes using ComputeSDK. Use when building applications that need Modal's GPU-accelerated sandbox environments for machine learning workloads, AI inference, or compute-intensive code execution.
---

# Modal Sandboxes with ComputeSDK

Run code on Modal's GPU-accelerated infrastructure through ComputeSDK's unified API. Modal provides on-demand GPU access and serverless containers — ideal for machine learning inference, training workloads, and compute-intensive applications.

## Setup

```bash
npm install computesdk
```

```bash
# .env
COMPUTESDK_API_KEY=your_computesdk_api_key
MODAL_TOKEN_ID=your_modal_token_id
MODAL_TOKEN_SECRET=your_modal_token_secret
```

Get your ComputeSDK key at https://console.computesdk.com/register

## Quick Start

```typescript
import { compute } from 'computesdk';
// Auto-detects Modal from environment variables

const sandbox = await compute.sandbox.create();

const result = await sandbox.runCode('import torch; print(torch.cuda.is_available())');
console.log(result.output);

await sandbox.destroy();
```

## Explicit Configuration

For multi-provider setups or when you want to be explicit:

```typescript
import { compute } from 'computesdk';

compute.setConfig({
  computesdkApiKey: process.env.COMPUTESDK_API_KEY,
  provider: 'modal',
  modal: {
    tokenId: process.env.MODAL_TOKEN_ID,
    tokenSecret: process.env.MODAL_TOKEN_SECRET,
  }
});

const sandbox = await compute.sandbox.create();
```

## Modal Configuration Options

```typescript
interface ModalConfig {
  tokenId?: string;             // Uses MODAL_TOKEN_ID env var if not set
  tokenSecret?: string;         // Uses MODAL_TOKEN_SECRET env var if not set
  runtime?: 'node' | 'python';  // Auto-detects from code patterns
  timeout?: number;              // Execution timeout in ms
  environment?: string;          // Modal environment ('sandbox' or 'main')
  ports?: number[];              // Ports to expose (unencrypted tunnels)
}
```

Ports are exposed with unencrypted tunnels by default for maximum compatibility.

## Full API

ComputeSDK provides the same API across all providers: filesystem operations, shell commands, managed servers, overlays, terminals, and client access. For the complete sandbox API, install the `computesdk` skill or see https://www.computesdk.com/docs/reference/sandbox/