# ComputeSDK Skills for AI Agents

A collection of skills that teach AI agents how to use [ComputeSDK](https://computesdk.com) — a unified TypeScript SDK for running code in remote sandboxes across multiple providers.

## What are Skills?

Skills are reusable capabilities for AI agents. Install them with a single command to give your agent procedural knowledge about ComputeSDK's API, provider setup, and sandbox management patterns.

## Available Skills

| Skill | Description |
|-------|-------------|
| [computesdk](./skills/computesdk) | Complete ComputeSDK reference — sandbox lifecycle, code execution, filesystem, managed servers, overlays, terminals, and client access |
| [e2b-sandbox](./skills/e2b-sandbox) | E2B provider setup — Firecracker microVMs with sub-second cold starts |
| [vercel-sandbox](./skills/vercel-sandbox) | Vercel provider setup — globally distributed serverless execution |
| [daytona-sandbox](./skills/daytona-sandbox) | Daytona provider setup — full development workspace environments |
| [modal-sandbox](./skills/modal-sandbox) | Modal provider setup — GPU-accelerated execution for ML workloads |
| [railway-sandbox](./skills/railway-sandbox) | Railway provider setup — self-hosted sandboxes on Railway infrastructure |
| [namespace-sandbox](./skills/namespace-sandbox) | Namespace provider setup — custom CPU/RAM allocation and architecture control |
| [render-sandbox](./skills/render-sandbox) | Render provider setup — self-hosted sandboxes with zero infrastructure setup |

## Installation

### 1. CLI Install (Recommended)

Install the main skill:

```bash
npx skills add https://github.com/computesdk/sandbox-skills --skill computesdk
```

Install a provider-specific skill:

```bash
npx skills add https://github.com/computesdk/sandbox-skills --skill e2b-sandbox
```

### 2. Claude Code — Project Skills

Clone into your project's `.claude/skills/` directory to share with your team via git:

```bash
git clone https://github.com/computesdk/sandbox-skills.git .claude/skills/sandbox-skills
```

### 3. Claude Code — Personal Skills

Clone into your personal skills directory for cross-project use:

```bash
git clone https://github.com/computesdk/sandbox-skills.git ~/.claude/skills/sandbox-skills
```

### 4. Manual Copy

Download the specific `SKILL.md` file you need and place it in your agent's skills directory.

## Usage

Once installed, your AI agent will automatically use these skills when you ask it to:

- "Set up a sandbox to run user code safely"
- "Create an E2B sandbox and execute Python code"
- "Build an app builder with live preview using ComputeSDK"
- "Set up a dev environment with managed servers and overlays"
- "Help me run untrusted code in an isolated environment"

## Skill Structure

```
sandbox-skills/
├── skills/
│   ├── computesdk/SKILL.md          # Main skill — full API reference
│   ├── e2b-sandbox/SKILL.md         # E2B provider guide
│   ├── vercel-sandbox/SKILL.md      # Vercel provider guide
│   ├── daytona-sandbox/SKILL.md     # Daytona provider guide
│   ├── modal-sandbox/SKILL.md       # Modal provider guide
│   ├── railway-sandbox/SKILL.md     # Railway provider guide
│   ├── namespace-sandbox/SKILL.md   # Namespace provider guide
│   └── render-sandbox/SKILL.md      # Render provider guide
└── README.md
```

The **main skill** (`computesdk`) covers the entire SDK API. The **provider skills** focus on provider-specific setup and configuration, and point back to the main skill for the full API reference.

## Links

- [ComputeSDK Documentation](https://www.computesdk.com/docs/)
- [ComputeSDK GitHub](https://github.com/computesdk/computesdk)
- [Get an API Key](https://console.computesdk.com/register)

## License

MIT