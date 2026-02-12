---
summary: "Step-by-step guide for setting up and updating API keys in OpenClaw"
read_when:
  - Setting up API keys for the first time
  - Updating or rotating existing API keys
  - Debugging authentication or missing API key errors
title: "API Key Setup"
---

# API Key Setup

This guide walks you through setting up or updating API keys in OpenClaw from scratch.

## Prerequisites

- OpenClaw installed (`npm i -g openclaw` or built from source)
- At least one model provider API key (Anthropic, OpenAI, Gemini, or OpenRouter)

## Quick start (interactive)

The fastest way to set up API keys is the onboarding wizard:

```bash
openclaw onboard
```

This walks you through provider selection, API key entry, and stores everything for daemon use.

## Step-by-step manual setup

### Step 1: Create the OpenClaw state directory

```bash
mkdir -p ~/.openclaw
```

This is where OpenClaw stores configuration, credentials, and session data.

### Step 2: Get your API key

Obtain an API key from one or more providers:

| Provider    | Where to get it                                                | Key format       |
| ----------- | -------------------------------------------------------------- | ---------------- |
| Anthropic   | [console.anthropic.com](https://console.anthropic.com)         | `sk-ant-...`     |
| OpenAI      | [platform.openai.com/api-keys](https://platform.openai.com)   | `sk-...`         |
| Gemini      | [aistudio.google.com](https://aistudio.google.com)             | alphanumeric     |
| OpenRouter  | [openrouter.ai/keys](https://openrouter.ai)                   | `sk-or-...`      |

### Step 3: Store the API key

Choose **one** of the following methods (from most recommended to least):

#### Method A: Global `.env` file (recommended for daemons)

```bash
cat >> ~/.openclaw/.env <<'EOF'
ANTHROPIC_API_KEY=sk-ant-your-key-here
OPENAI_API_KEY=sk-your-key-here
EOF
```

This is the best option if you run OpenClaw as a systemd/launchd daemon, because the daemon process reads this file on startup.

#### Method B: Local `.env` file (for development)

Create a `.env` in your working directory (the directory you run `openclaw` from):

```bash
cat >> .env <<'EOF'
ANTHROPIC_API_KEY=sk-ant-your-key-here
EOF
```

This takes higher precedence than `~/.openclaw/.env` but only works when you launch from that directory.

#### Method C: Config file `env` block

Edit `~/.openclaw/openclaw.json` (JSON5):

```json5
{
  env: {
    ANTHROPIC_API_KEY: "sk-ant-your-key-here",
    OPENAI_API_KEY: "sk-your-key-here",
  },
}
```

This is lower precedence than `.env` files — useful as a fallback.

#### Method D: Shell export (ephemeral)

```bash
export ANTHROPIC_API_KEY="sk-ant-your-key-here"
openclaw gateway
```

Highest precedence, but lost when the shell session ends.

### Step 4: Verify the key works

```bash
openclaw models status
```

This shows which providers have valid credentials. You should see your provider listed with an active status.

Run the full diagnostic:

```bash
openclaw doctor
```

This checks configuration, credentials, connectivity, and reports any issues.

### Step 5: Restart the Gateway (if already running)

If the Gateway was already running when you changed keys:

```bash
openclaw gateway restart
```

Or, if using a daemon:

```bash
openclaw service restart
```

## Updating an existing API key

### Option 1: Edit the `.env` file directly

```bash
# Edit the global env file
nano ~/.openclaw/.env
# or
vim ~/.openclaw/.env
```

Find the line with your key and replace the value:

```
ANTHROPIC_API_KEY=sk-ant-new-key-here
```

Then restart:

```bash
openclaw gateway restart
```

### Option 2: Use the CLI

```bash
openclaw config set env.ANTHROPIC_API_KEY "sk-ant-new-key-here"
```

The Gateway hot-reloads config changes automatically (no restart needed for most settings).

### Option 3: Re-run onboarding

```bash
openclaw onboard
```

The wizard detects existing configuration and lets you update individual values.

## Setting up gateway authentication

If the Gateway binds beyond loopback (e.g., accessible from other devices), set a gateway token:

```bash
# Generate a random token
TOKEN=$(openssl rand -hex 32)

# Store it
cat >> ~/.openclaw/.env <<EOF
OPENCLAW_GATEWAY_TOKEN=$TOKEN
EOF
```

Or set a password instead:

```bash
cat >> ~/.openclaw/.env <<'EOF'
OPENCLAW_GATEWAY_PASSWORD=your-strong-password-here
EOF
```

## Environment variable precedence

When the same variable is defined in multiple places, highest wins:

1. **Process environment** (shell export)
2. **`.env` in current working directory**
3. **`~/.openclaw/.env`** (global)
4. **`openclaw.json` `env` block**
5. **Shell env import** (if `OPENCLAW_LOAD_SHELL_ENV=1`)

Existing non-empty values are never overridden by lower-precedence sources.

## Troubleshooting

### "No credentials found"

```bash
openclaw models status    # check which providers are detected
openclaw doctor           # full diagnostic
```

Verify the key is in one of the expected locations and the variable name is correct (e.g., `ANTHROPIC_API_KEY`, not `ANTHROPIC_KEY`).

### Key works in shell but not in daemon

The daemon may not inherit your shell environment. Either:

- Put the key in `~/.openclaw/.env`
- Or enable shell env import: `OPENCLAW_LOAD_SHELL_ENV=1`

### Token expired (subscription auth)

If using `claude setup-token` instead of an API key:

```bash
claude setup-token                                          # regenerate
openclaw models auth setup-token --provider anthropic       # import
openclaw models status                                      # verify
```

## Related

- [Environment variables](/help/environment)
- [Authentication](/gateway/authentication)
- [Configuration](/gateway/configuration)
- [Doctor](/gateway/doctor)
