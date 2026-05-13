---
id: clawboard-required-config
title: Key openclaw.json Settings for Clawboard
sidebar_label: Clawboard Required Config
sidebar_position: 4
---

Clawboard API connects to OpenClaw via two services: the workspace service (port 18780) and the gateway
(port 18789). Certain `openclaw.json` settings must be configured correctly for Clawboard features to
work. This page lists the settings that matter most.

:::tip For a complete reference of every field, see the [openclaw.json Reference](./openclaw-json).
For a complete annotated example, see the [Sample Configuration](./sample-config). :::

---

## Gateway: allowed origins

**Required for the dashboard to communicate with the gateway.**

```json
{
  "gateway": {
    "controlUi": {
      "allowedOrigins": [
        "http://localhost:18789",
        "https://your-openclaw-domain.example.com",
        "https://your-clawboard-dashboard.example.com"
      ]
    }
  }
}
```

Add both your OpenClaw UI origin and your Clawboard dashboard origin. If the dashboard origin is
missing, the browser will block WebSocket and API calls to the gateway with a CORS error.

---

## Gateway: pairing prerequisites

**Required for Clawboard's device pairing workflow.**

```json
{
  "gateway": {
    "controlUi": {
      "allowedOrigins": [
        "http://localhost:18789",
        "http://127.0.0.1:18789",
        "https://your-openclaw-domain.example.com"
      ]
    }
  }
}
```

Clawboard API now uses the dashboard pairing wizard to provision a device-authenticated gateway
identity. For that flow to work:

- `gateway.auth.mode` must be `token`
- `gateway.controlUi.allowedOrigins` must include the actual gateway origin Clawboard opens
- `OPENCLAW_GATEWAY_TOKEN` must be configured in Clawboard API
- An `owner` or `admin` must complete `Settings -> OpenClaw Pairing`

Clawboard no longer relies on `allowInsecureAuth` as an operational fallback for gateway RPCs.

---

## Gateway: authentication mode

**Required for gateway token bootstrap auth.**

```json
{
  "gateway": {
    "auth": {
      "mode": "token"
    }
  }
}
```

Set `auth.mode: "token"` so Clawboard API can bootstrap the pairing handshake using the
`OPENCLAW_GATEWAY_TOKEN` environment variable.

---

## Tools: session visibility

**Required for session data to appear in the Agent Monitor.**

```json
{
  "tools": {
    "sessions": {
      "visibility": "agent"
    }
  }
}
```

`visibility: "agent"` allows agents to see their own session history. Without this, the
`sessions_list` and `sessions_history` tool calls that Clawboard uses will return empty results.

---

## Tools: agent-to-agent

**Required for subagent features.**

```json
{
  "tools": {
    "agentToAgent": {
      "enabled": true
    }
  }
}
```

Enable this if you want agents to delegate work to other agents as subagents. Clawboard's subagent
activity tracking depends on this being enabled.

---

## Agents: workspace paths

**Required for the workspace file browser and agents page.**

Each agent must have a `workspace` path that points to a real directory on the workspace filesystem:

```json
{
  "agents": {
    "list": [
      {
        "id": "coo",
        "default": true,
        "workspace": "/home/node/.openclaw/workspace-coo"
      }
    ]
  }
}
```

Clawboard API reads this path from `openclaw.json` via the workspace service to determine where each
agent's files live. If the path is missing or wrong, the workspace browser will show an empty
directory or a 404.

---

## Agents: identity

**Required for the agents page and agent selector.**

```json
{
  "agents": {
    "list": [
      {
        "id": "coo",
        "identity": {
          "name": "Clawboard",
          "theme": "Research - Delegation - Execution - Orchestration",
          "emoji": "🤖"
        }
      }
    ]
  }
}
```

Clawboard reads `identity.name` and `identity.emoji` to display agents in the agents page and workspace
selector. Without these, agents will show as their raw `id` with no icon.

---

## Memory: shared docs path

**Recommended for agents to access shared documentation.**

```json
{
  "memory": {
    "backend": "qmd",
    "qmd": {
      "includeDefaultMemory": true,
      "paths": [
        {
          "path": "../docs",
          "name": "shared-docs",
          "pattern": "**/*.md"
        }
      ]
    }
  }
}
```

The `../docs` path (relative to each agent's workspace) points to the shared `docs/` directory at
the workspace root. This makes shared documentation available as memory to all agents.

:::info Workspace path convention Clawboard expects shared content at the workspace root:

```text
/                         ← workspace root
├── workspace-coo/        ← agent workspace
├── workspace-cto/        ← agent workspace
├── docs/                 ← shared docs (memory source)
├── projects/             ← shared projects
└── openclaw.json
```

:::

---

## Channels: Telegram bot token

**Required if using Telegram.**

```json
{
  "channels": {
    "telegram": {
      "enabled": true,
      "accounts": {
        "default": {
          "botToken": "${TELEGRAM_BOT_TOKEN}"
        }
      }
    }
  },
  "plugins": {
    "entries": {
      "telegram": {
        "enabled": true
      }
    }
  }
}
```

Always reference the bot token via `${TELEGRAM_BOT_TOKEN}` — never hardcode it. The matching
`plugins.entries.telegram.enabled: true` is also required to activate Telegram support.

---

## Device pairing workflow

**Required for full gateway-backed Clawboard features.**

Device auth uses an Ed25519 key pair to authenticate Clawboard API as a trusted device. Once paired,
Clawboard receives the operator scopes it needs for session visibility, usage, and runtime control.

### Step 1: Configure gateway bootstrap

Add these values to Clawboard API's `.env`:

```bash
OPENCLAW_GATEWAY_URL=http://localhost:18789
OPENCLAW_GATEWAY_TOKEN=your-gateway-token
```

### Step 2: Start pairing in Clawboard

1. Restart Clawboard API
2. Sign in as an `owner` or `admin`
3. Open `Settings -> OpenClaw Pairing`
4. Click `Start pairing`

### Step 3: Approve and finalize

1. Approve the pending Clawboard device in OpenClaw
2. Return to Clawboard and click `Finalize pairing`
3. Confirm the integration status is `ready`

---

## Minimal openclaw.json for Clawboard

The smallest `openclaw.json` that gives Clawboard full functionality:

```json
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "openrouter/anthropic/claude-sonnet-4.6",
        "fallbacks": []
      }
    },
    "list": [
      {
        "id": "agent",
        "default": true,
        "workspace": "/home/node/.openclaw/workspace-agent",
        "identity": {
          "name": "Agent",
          "theme": "General Assistant",
          "emoji": "🤖"
        }
      }
    ]
  },
  "tools": {
    "sessions": {
      "visibility": "agent"
    },
    "agentToAgent": {
      "enabled": true
    }
  },
  "gateway": {
    "port": 18789,
    "mode": "local",
    "bind": "lan",
    "controlUi": {
      "allowedOrigins": ["http://localhost:18789", "https://your-clawboard-dashboard.example.com"]
    },
    "auth": {
      "mode": "token"
    },
    "tls": {
      "enabled": false
    }
  },
  "memory": {
    "backend": "qmd",
    "qmd": {
      "includeDefaultMemory": true
    }
  }
}
```

---

## Quick reference

| Setting                                 | Required for                      | Default if missing     |
| --------------------------------------- | --------------------------------- | ---------------------- |
| `gateway.controlUi.allowedOrigins`      | Dashboard → gateway communication | CORS errors            |
| `gateway.controlUi.allowedOrigins`      | Pairing WebSocket origin checks   | Origin mismatch errors |
| `gateway.auth.mode: "token"`            | Bearer token auth                 | Auth may fail          |
| `tools.sessions.visibility: "agent"`    | Agent Monitor session data        | Empty session list     |
| `tools.agentToAgent.enabled`            | Subagent tracking                 | Subagents not tracked  |
| `agents[].workspace`                    | Workspace browser, agents page    | 404 / empty workspace  |
| `agents[].identity`                     | Agents page display names         | Raw IDs, no icons      |
| `memory.qmd.paths`                      | Shared docs in agent memory       | No shared memory       |
| `channels.telegram.accounts[].botToken` | Telegram integration              | Telegram disabled      |
| `plugins.entries.telegram.enabled`      | Telegram plugin activation        | Telegram disabled      |
