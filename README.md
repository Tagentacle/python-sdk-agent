# python-sdk-agent

**Agent SDK** — Composable building blocks for Tagentacle AI agent nodes.

## Overview

This package provides **mechanisms** (not policies) for building agent nodes:

| Component | Description |
|-----------|-------------|
| `MessageQueue` | Message buffer between inference cycles (~30 lines) |
| `InferenceMux` | Controls when to infer: IDLE/BUSY state machine + priority queue (~50 lines) |
| `ContextFactory` | `mailbox + messages + tools → LLMRequest` pure function (~40 lines) |

### Design Principles

- **All components are optional** — pick what you need
- **No topic names hardcoded** — SDK provides event hooks, topic binding is in bringup config
- **No policies** — InferenceMux emits events (`on_inference_start`, `on_tool_call`, `on_complete`), it doesn't decide where to publish them
- **< 500 lines total** — if it exceeds this, policies are leaking in

### Event Hooks (Q13)

```python
class InferenceMux:
    # Events — consumers decide what to do with them
    on_inference_start: Callable
    on_tool_call: Callable
    on_inference_complete: Callable

# AgentNode (example-agent, NOT this SDK) decides:
async def handle_tool_call(event):
    topic = config["trace_topic"]  # from bringup config
    if topic:
        await node.publish(topic, event.to_dict())
```

## Architecture Context

```
python-sdk-core:    Node, LifecycleNode (kernel-level)
python-sdk-mcp:     MCPServerComponent, TagentacleMCPServer (MCP layer)
python-sdk-agent:   MessageQueue, InferenceMux, ContextFactory (agent building blocks)  ← this
python-sdk-tacl:    TACLAuthority, TACLMiddleware, auth (access control)

example-agent:      AgentNode = LifecycleNode + agent SDK components + policy choices
```

## Status

**Phase 1 — full-stack-v1**: Package created, components to be extracted from `example-agent` design.

→ [Project board](https://github.com/orgs/Tagentacle/projects/1)

## License

MIT
