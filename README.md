# sandbox-skills

ComputeSDK provider skills and reference guides for Devin.

## Available Skills

| Skill | Description |
|-------|-------------|
| `computesdk` | Universal ComputeSDK sandbox API reference and multi-provider setup |
| `agentcore-sandbox` | AWS Bedrock AgentCore Code Interpreter provider for ComputeSDK - secure, session-based code execution sandboxes |
| `agentuity-sandbox` | Agentuity provider for ComputeSDK - isolated cloud sandboxes with native filesystem, snapshot/checkpoint support, and flexible runtimes |
| `archil-sandbox` | Archil provider for ComputeSDK - exec commands against an Archil disk |
| `arker-sandbox` | Arker provider for ComputeSDK - sandboxed VMs with persistent filesystems, forked from golden images |
| `beam-sandbox` | Beam provider for ComputeSDK - containerized sandbox environments with process management and filesystem access |
| `blaxel-sandbox` | Blaxel provider for ComputeSDK - lightweight cloud sandboxes for code execution |
| `cloud-run-sandbox` | Google Cloud Run Sandboxes provider for ComputeSDK |
| `cloudflare-sandbox` | Cloudflare provider for ComputeSDK - edge code execution using Cloudflare Workers and Durable Objects |
| `codesandbox-sandbox` | CodeSandbox provider for ComputeSDK - fast browser-compatible sandboxes with npm and Python support |
| `collimate-sandbox` | Collimate provider for ComputeSDK |
| `createos-sandbox` | CreateOS provider for ComputeSDK — NodeOps VM sandboxes with pause/resume/fork snapshots |
| `daytona-sandbox` | Daytona provider for ComputeSDK - standardized development environments with devcontainer support |
| `declaw-sandbox` | Declaw provider for ComputeSDK - secure sandboxes with PII scanning, prompt-injection defense, and network egress filtering |
| `docker-sandbox` | Docker provider for ComputeSDK - local containerized sandboxes for development and testing |
| `e2b-sandbox` | E2B provider for ComputeSDK - cloud sandboxes with full Linux environments, filesystem access, and microVM isolation |
| `freestyle-sandbox` | Freestyle provider for ComputeSDK - cloud sandboxes powered by Freestyle |
| `hopx-sandbox` | HopX provider for ComputeSDK - cloud sandboxes with full Linux environments, filesystem access, and microVM isolation |
| `isorun-sandbox` | Isorun provider for ComputeSDK — isolated Linux VM sandboxes for running untrusted and AI-generated code |
| `just-bash-sandbox` | just-bash provider for ComputeSDK - local sandboxed bash execution with virtual filesystem |
| `k8s-sandbox` | Kubernetes provider for ComputeSDK - run sandboxes as pods |
| `leap0-sandbox` | Leap0 provider for ComputeSDK - cloud sandboxed environments for AI agents |
| `lelantos-sandbox` | Lelantos provider for ComputeSDK - EU-native Firecracker microVM sandboxes (E2B-API-compatible) with full Linux environments, filesystem access, and per-port preview URLs |
| `lightning-sandbox` | Lightning AI provider for ComputeSDK - cloud sandboxes for code execution, command running, and filesystem access |
| `modal-sandbox` | Modal provider for ComputeSDK - serverless Python execution with GPU support and zero cold starts |
| `mosaic-sandbox` | Mosaic provider for ComputeSDK - Firecracker-based sandbox environments |
| `namespace-sandbox` | Namespace provider for ComputeSDK - cloud-native sandboxes with optional GPU support |
| `neevcloud-sandbox` | NeevCloud provider for ComputeSDK - secure cloud sandboxes with command execution, filesystem access, and preview URLs |
| `northflank-sandbox` | Northflank provider for ComputeSDK - Deploy and manage compute workloads on Northflank's container platform |
| `opencomputer-sandbox` | OpenComputer provider for ComputeSDK - persistent cloud VMs with checkpoints, preview URLs, command execution, and filesystem access |
| `quilt-sandbox` | Quilt provider for ComputeSDK - tenant-scoped Linux sandboxes with exec, published services, and snapshots |
| `railway-sandbox` | Railway Sandboxes provider for ComputeSDK - run commands in Railway-hosted sandboxes |
| `run-cloud-sandbox` | Run Cloud provider for ComputeSDK - fast Firecracker microVM sandboxes with snapshots and filesystem access |
| `runloop-sandbox` | Runloop provider for ComputeSDK - AI-optimized code execution with built-in devtools and debugging |
| `sail-sandbox` | Sail provider for ComputeSDK - fast, isolated microVM sandboxes with native filesystem access |
| `sandbox0-sandbox` | Sandbox0 provider for ComputeSDK - fast persistent sandboxes with command execution and native filesystem access |
| `secure-exec-sandbox` | Secure execution provider for ComputeSDK - isolated sandbox environments using secure-exec |
| `sprites-sandbox` | Sprites provider for ComputeSDK - cloud sandboxes powered by Sprites |
| `superserve-sandbox` | Superserve provides sandbox infrastructure to run code in isolated cloud environments powered by Firecracker MicroVMs |
| `tenki-sandbox` | Tenki Cloud provider for ComputeSDK - microVM sandboxes with native filesystem, preview URLs, snapshots, and SSH |
| `tensorlake-sandbox` | Tensorlake provider for ComputeSDK - stateful MicroVM sandboxes for agentic applications and LLM-generated code execution |
| `upstash-sandbox` | Upstash Box provider for ComputeSDK - cloud sandboxes with code execution, filesystem access, and AI agent support |
| `vercel-sandbox` | Vercel Sandbox provider for ComputeSDK - serverless code execution for Python and Node.js on Vercel's edge network |

## Usage

Install a provider skill with Devin's `skills` command:

```bash
npx skills add https://github.com/computesdk/sandbox-skills --skill e2b-sandbox
```

Or use the main ComputeSDK skill for the universal API reference:

```bash
npx skills add https://github.com/computesdk/sandbox-skills --skill computesdk
```

## Contributing

Skills are generated from the [ComputeSDK monorepo](https://github.com/computesdk/computesdk). Update the source documentation there, then regenerate these skill files.

## License

MIT
