# python-sdk-agent

**Agent SDK** — Composable building blocks for Tagentacle AI agent nodes.

## Overview

This package provides **mechanisms** (not policies) for building agent nodes:

| Component | Description |
|-----------|-------------|
| `Inbox` | Agent-local message buffer with per-topic attention levels (~30 lines) |
| `InferenceMux` | Controls when to infer: IDLE/BUSY state machine + followup queue (~50 lines) |

> **Removed**: `MessageQueue` (bus callback → Inbox replaces it), `ContextFactory` (context assembly is policy, not mechanism — stays in example-agent)

### Design Principles

- **All components are optional** — pick what you need
- **No topic names hardcoded** — SDK doesn’t know any specific topic
- **No policies** — Inbox buffers, InferenceMux triggers; example-agent decides what to do
- **< 100 lines total** — if it exceeds this, policies are leaking in
- **Attention levels are a mixin** — remove classification and Inbox degrades to a plain list buffer

### Two-Plane Model (Q14 revised)

```
Agent = LifecycleNode (bus data plane) + Multi-MCP Client (tool plane)

Bus (data):  subscribe topics, call services, publish results
MCP (tools): connect to shell-server, weather-server, etc.
```

### Message Flow

```
Bus topic callback → Inbox.push(topic, msg)
                       ├─ mode=followup → buffer + signal InferenceMux
                       └─ mode=collect  → buffer only (silent)

InferenceMux.trigger() → _agentic_loop() reads Inbox.drain()
```

## Architecture Context

```
python-sdk-core:    Node, LifecycleNode (kernel-level)
python-sdk-mcp:     MCPServerComponent, TagentacleMCPServer (MCP layer)
python-sdk-agent:   Inbox, InferenceMux (agent building blocks)  ← this
python-sdk-tacl:    TACLAuthority, TACLMiddleware, auth (access control)

example-agent:      AgentNode = LifecycleNode + agent SDK components + policy choices
```

## Status

**Phase 1 — full-stack-v1**: Package created, components to be extracted from `example-agent` design.

→ [Project board](https://github.com/orgs/Tagentacle/projects/1)

## License

MIT
