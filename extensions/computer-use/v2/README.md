# Native Computer Use HTTP

This guest extension describes the locally verified `rust-envd` implementation from `feat/native-embedded-cua`. The `v2` label versions this document. HTTP paths are rooted at `/cua` on the envd origin. [OpenAPI](openapi.yml)

- `GET /cua/tools` discovers tool names, descriptions and `input_schema`.
- `POST /cua/call` runs one tool using `name` and object-valued `arguments`. The JSON response carries `success`, `content` and `data`. Tool-reported failures use `success: false` with driver diagnostics in `data`. Image items identify their media type through `mime_type`.
- `GET /cua/health` reports passive runtime availability with HTTP `200` or `503`. `GET /cua/metrics` exposes Prometheus metrics.

CUA requests use the initialized envd instance's current `X-Access-Token`. Rotate the token using the existing envd lifecycle. Authentication applies before tool invocation. A desktop-free build or a Windows service without an eligible user session can report an unavailable runtime. Probe availability with `GET /cua/health` using an authorized token.

```http
POST /cua/call HTTP/1.1
X-Access-Token: <envd-access-token>
Content-Type: application/json

{"name":"get_screen_size","arguments":{}}
```

An illustrative successful tool result is `{"success":true,"content":[{"type":"text","text":"Screen size available"}],"data":null}`. Tool-specific content comes from the selected desktop driver. A transport error uses the existing envd JSON envelope with `code` and `message`.

Requests are capped at 1 MiB and serialized responses at 32 MiB. The execution deadline is 30 seconds. An HTTP timeout or disconnect can leave the underlying desktop action running and holding the execution slot. The native implementation rejects MCP JSON-RPC bodies at `/cua/call`. Clients using [MCP discovery and invocation](../v1/README.md) require a deployment that implements those interfaces.
