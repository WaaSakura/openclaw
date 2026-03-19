# AgentManager remote MCP setup

This page shows how to connect OpenClaw to an AgentManager MCP server running on the network.

## Quick path

If you already know the AgentManager host and token, add a remote server entry under `mcp.servers.<name>.url`:

```json
{
  "mcp": {
    "servers": {
      "agentmanager": {
        "url": "http://127.0.0.1:4173/mcp",
        "headers": {
          "Authorization": "Bearer ${AGENTMANAGER_MCP_TOKEN}"
        }
      }
    }
  }
}
```

Then load it with `/mcp set`:

```text
/mcp set agentmanager={"url":"http://127.0.0.1:4173/mcp","headers":{"Authorization":"Bearer ${AGENTMANAGER_MCP_TOKEN}"}}
```

Recommended next check:

- read `agentmanager://system/mcp-principal`
- confirm the token's `scopes` and `dangerousAllowed` flags match what you intended

## What you are connecting to

- AgentManager exposes a Streamable HTTP MCP server at `/mcp`.
- The server is intended for operator-side, control-plane access over a trusted local network.
- Authentication uses a bearer token, not the OpenClaw session cookie.

## Recommended OpenClaw config

Use a URL-based MCP server entry under `mcp.servers`:

```json
{
  "mcp": {
    "servers": {
      "agentmanager": {
        "url": "http://127.0.0.1:4173/mcp",
        "headers": {
          "Authorization": "Bearer ${AGENTMANAGER_MCP_TOKEN}"
        }
      }
    }
  }
}
```

Recommended notes:

- Keep the token in an environment variable or secret store.
- Replace `127.0.0.1` with the AgentManager host IP or DNS name when connecting over LAN.
- Use the same bearer token you configured for AgentManager `MCP_TOKEN`.

## Minimal validation flow

1. Confirm unauthenticated access is rejected.

```bash
curl -i http://127.0.0.1:4173/mcp
```

Expected:

- `401 Unauthorized`

2. Confirm the MCP client can list resources with the bearer token.

Expected resources include:

- `agentmanager://system/health`
- `agentmanager://system/report`
- `agentmanager://system/operator-queue`
- `agentmanager://system/governance-board`
- `agentmanager://system/mcp-principal`
- `agentmanager://projects`

3. Read `agentmanager://system/health` and confirm the control-plane summary is returned.

4. Call a safe tool, such as:

- `system.listBlockedTasks`
- `project.start`
- `task.create`

5. Call a dangerous tool without `reason`, and confirm it is rejected.

- `dangerous.taskRollback`
- `dangerous.projectDelete`

## Why `url` instead of `command`

OpenClaw `mcp.servers` supports both local `command` servers and remote `url` servers.

Use `url` when:

- the MCP server is already running on the network, such as AgentManager on a LAN host
- you want OpenClaw's main control plane to talk to that server directly
- you do not want to manage a separate local MCP subprocess

Use `command` when:

- the MCP server must be spawned locally by OpenClaw
- the tool provider is only available as a local process

## Supported boundary

This setup is intended for OpenClaw main control-plane or owner-side integrations.

Some embedded or runtime-adapter paths in OpenClaw may still be stdio-only today. Those paths are not the primary target of this guide.

## AgentManager capabilities you can expect

Read-only resources:

- system health and report snapshots
- operator queue and governance board views
- project, task, run, and agent resources

Tools:

- safe project / task / runtime operations
- dangerous operations that require `reason`

Run observability:

- resolved runtime config
- provider / model
- runtime source and override metadata

Principal self-description:

- current token name
- scopes
- dangerous permission
- allowed resource prefixes / tool groups

## Related docs

- [Slash Commands](/docs/tools/slash-commands.md)
- AgentManager-side MCP operator docs live in the AgentManager repository (`docs/mcp-control-plane.md`).
