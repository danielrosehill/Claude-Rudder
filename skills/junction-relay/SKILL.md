---
name: junction-relay
description: "[EXPERIMENTAL] Use Agent Junction to relay context between two running Claude Code sessions in real time. Requires Agent Junction MCP server running on localhost or LAN. Use when the user wants two agents to talk to each other, share findings, or coordinate work across repos."
---

# Junction Relay (Experimental)

Establish a live relay between this Claude Code session and another via [Agent Junction](https://github.com/danielrosehill/Agent-Junction) — an encrypted peer-to-peer message bus over MCP.

**Status**: Experimental. Requires Agent Junction running and configured as an MCP server in both sessions.

## When to use

- Two Claude sessions are working on related tasks and need to share information
- One session has context (config, paths, decisions) the other needs
- Coordinating work across repos without the human manually relaying messages

## Prerequisites

1. Agent Junction server is running: `npx agent-junction` (or `agent-junction` if installed globally)
2. Both Claude Code sessions have Junction configured in `.mcp.json` or `~/.claude/settings.json`:
   ```json
   {
     "mcpServers": {
       "junction": {
         "type": "streamable-http",
         "url": "http://127.0.0.1:4200/mcp"
       }
     }
   }
   ```

## Steps

### 1. Check Junction is available

```bash
curl -s http://127.0.0.1:4200/health
```

If this fails, tell the user: "Agent Junction isn't running. Start it with `npx agent-junction` in a separate terminal, then try again."

### 2. Register this session

Use the Junction MCP `register` tool with:
- `repo`: current working directory
- `task`: brief description of what this session is doing
- `role`: `"relay-sender"` (or `"relay-receiver"` if resuming from a peer)

Note the alias you receive (e.g. `crimson-falcon`). Tell the user your alias.

### 3. Discover peer

Use `list_peers` to see connected sessions. If no peer is connected yet, tell the user to open or configure the other session, then retry.

### 4. Exchange context

**Sending**: Use `send_message` with the peer's alias and the context to relay. Structure messages as actionable context:
```
RELAY FROM [your-alias] @ [repo-name]
---
[The context, decision, finding, or request]
---
ACTION NEEDED: [what the peer should do with this, or "FYI only"]
```

**Receiving**: Use `read_messages` to check for incoming messages. Messages are read-once (deleted after reading), so capture the content before proceeding.

### 5. Report

Tell the user:
- Your alias and the peer's alias
- What was sent or received
- Whether the relay is still active (both peers connected)

## Notes

- Messages are encrypted with AES-256-GCM and deleted after reading
- Junction is ephemeral — all data is purged when peers disconnect
- This is a point-to-point relay, not a persistent channel. Each message exchange is explicit.
- For LAN relay (sessions on different machines), the Junction server must bind to `0.0.0.0` via `JUNCTION_HOST=0.0.0.0`
- If Junction isn't available, fall back to file-based handover (`/write-handover` + `/start-from-handover`)
