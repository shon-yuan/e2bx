# E2BX Sandbox Protocol

English | [简体中文](README.zh-CN.md)

E2BX is an independent protocol extension project based on the E2B sandbox interfaces. This repository contains machine-readable OpenAPI and Protobuf definitions for client and server implementers. The initial baseline comes from [e2b-dev/runtime commit `391b22443460fb30432da889b44aa413478d5717`](https://github.com/e2b-dev/runtime/tree/391b22443460fb30432da889b44aa413478d5717). E2BX-specific extensions are drafted in [extensions/](extensions/README.md).

## Four specifications used by SDKs

| Operations | Specification |
| --- | --- |
| Create, connect to, pause, and terminate sandboxes; manage templates and snapshots | [Control plane REST](upstream/runtime/spec/openapi.yml) |
| Upload and download files; use envd HTTP interfaces | [envd HTTP](upstream/runtime/packages/envd/spec/envd.yaml) |
| Start processes, receive output, write to stdin, connect to processes, and manage PTYs | [Process RPC](upstream/runtime/packages/envd/spec/process/process.proto) |
| Query attributes, operate on directories, and watch for file changes | [Filesystem RPC](upstream/runtime/packages/envd/spec/filesystem/filesystem.proto) |

To run a command, a client first obtains the `sandboxID`, domain, and envd credentials from the control plane, then calls `/process.Process/Start` on that sandbox. Process results are returned as an event stream, and generated files can be downloaded through `GET /files`. The REST path version and a template's `envdVersion` describe the control plane interface and guest capabilities, respectively. The control plane specification also includes administrative endpoints; envd HTTP operations marked `x-internal: true` are intended for the host.

## Other contracts in the same baseline

| Scope | Definitions |
| --- | --- |
| Edge and Hyperloop HTTP services | [Edge](upstream/runtime/spec/openapi-edge.yml), [Hyperloop](upstream/runtime/spec/openapi-hyperloop.yml) |
| Inter-node calls and template builds | [Orchestrator, Info, Chunks, Template Manager, Volume](upstream/runtime/packages/orchestrator/) |
| Proxy-to-control-plane communication | [Proxy RPC](upstream/runtime/packages/shared/pkg/grpc/proxy/proxy.proto) |
| Scheduling service | [Reservation and State](upstream/runtime/packages/scheduling-api/e2b/scheduling/v1/) |
| Legacy guest interfaces and upgrade handover | [Legacy Process / Filesystem](upstream/runtime/packages/envd/internal/services/legacy/), [Handover](upstream/runtime/packages/envd/spec/upgrade/handover.proto) |

## Verification and updates

`upstream/runtime/` retains the upstream paths. The initial import contains the four OpenAPI definitions and all 13 `.proto` files from that commit. The root `LICENSE` is the original upstream Apache-2.0 license. [SHA256SUMS](SHA256SUMS) records the byte digests of those original files and the license. From the repository root, run:

```sh
sha256sum -c SHA256SUMS
```

When reading the definitions with `protoc`, use `packages/envd/spec` as the import root for guest protocols. Node and proxy protocols use the `packages/orchestrator` and `packages/shared/pkg/grpc/proxy` import roots. These paths are relative to `upstream/runtime/`. Standard types are provided by Protobuf. The scheduling protocol also depends on `buf/validate/validate.proto`, pinned upstream to `buf.build/bufbuild/protovalidate:511051f7f4374c3ca873b53ae68a9288`. See the [upstream generation instructions](https://github.com/e2b-dev/runtime/blob/391b22443460fb30432da889b44aa413478d5717/packages/scheduling-api/generate.go#L3-L4).

When updating the upstream baseline, update the commit hash, source links, specification files, and digests together. For cross-interface behavior such as SDK routing, authentication, and runtime errors, refer to the [architecture documentation](https://github.com/e2b-dev/runtime/blob/391b22443460fb30432da889b44aa413478d5717/docs/ARCHITECTURE.md) at the same commit.
