# Computer Use over MCP

This guest extension describes the rust-envd integrated desktop interface recorded at `0155255ddd73bf5ed7bcc9cbe45f309eaf3e384c`. The paths below are relative to the envd origin. [OpenAPI](openapi.yml)

1. Read `GET /cua/info`. Look for `transport: mcp-jsonrpc-http` and use the same-origin `proxy_path` when advertised. A deployment may advertise a direct `mcp_url` that only works from the worker's own network namespace.
2. Connect an MCP Streamable HTTP client. Initialize, call `tools/list` to discover input schemas and use `tools/call` to invoke a tool. An example tool is `get_desktop_state`.

After MCP initialization, a tool discovery request can look like this. Supply any session or protocol headers negotiated by the MCP client.

```http
POST /cua/mcp HTTP/1.1
Authorization: Bearer <cua-credential>
Content-Type: application/json
Accept: application/json, text/event-stream

{"jsonrpc":"2.0","id":2,"method":"tools/list"}
```

An abbreviated JSON reply shows the tool descriptor fields.

```json
{"jsonrpc":"2.0","id":2,"result":{"tools":[{"name":"get_desktop_state","inputSchema":{"type":"object"}}]}}
```

Use the complete `inputSchema` advertised by the server when constructing a `tools/call` request. In `embedded` and `shared-external` modes the Bearer credential is the current envd access token. In legacy proxy mode it is the configured proxy token. `GET /cua/health` and `GET /cua/metrics` expose deployment health and Prometheus metrics. The separate [native HTTP contract](../v2/README.md) uses different routes and admission semantics.
