# mcp

The [MCP Python SDK](https://github.com/modelcontextprotocol/python-sdk) implements
both sides of the Model Context Protocol, so it owns the MCP client and server
operations and makes no model calls itself. The scenario runs an SDK client
against an SDK server ([server.py](server.py)) over stdio and records the
operation metrics in both processes from each JSON-RPC message where the SDK
hands it to, or takes it from, the transport.

| Operation | Should be instrumented here | Status |
| --- | --- | --- |
| MCP client operation metrics (`mcp.client.operation.duration`, `mcp.client.operation.active`, `mcp.client.request.size`, `mcp.client.response.size`) | Yes — the SDK sends the requests and notifications | ✅ Implemented |
| MCP server operation metrics (`mcp.server.operation.duration`, `mcp.server.operation.active`, `mcp.server.request.size`, `mcp.server.response.size`) | Yes — the SDK receives and answers the requests | ✅ Implemented for requests; nothing answers a notification on stdio, so only its sender records it |
| MCP session metrics (`mcp.client.session.duration`, `mcp.server.session.duration`) | Yes — the SDK owns the session | ❌ Not implemented |
| MCP client and server spans | Yes — the SDK sends and handles every message | ❌ Not implemented |

The SDK emits its own spans on the `mcp-python-sdk` tracer; the scenario drops
them. Its server spans carry `gen_ai.operation.name=execute_tool`, so they
would be counted as GenAI `execute_tool` spans and fail that span's name check.

On stdio there is no server address or application-layer protocol, so
`server.address`, `server.port`, and `network.protocol.*` are not set, and the
SDK only speaks JSON-RPC 2.0, so `jsonrpc.protocol.version` is not set either.
The opt-in `mcp.resource.uri` has no resource operation to describe here.
