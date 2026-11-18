# E2BX 沙箱协议

E2BX 是基于 E2B 沙箱接口的独立协议扩展项目。本仓库收录机器可读的 OpenAPI 与 Protobuf 定义，供客户端和服务端实现者引用。首个基线来自 [e2b-dev/runtime 的提交 `391b22443460fb30432da889b44aa413478d5717`](https://github.com/e2b-dev/runtime/tree/391b22443460fb30432da889b44aa413478d5717)。E2BX 自有扩展从 [extensions/](extensions/README.md) 起草。

## SDK 使用的四份规范

| 要实现的操作 | 规范 |
| --- | --- |
| 创建、连接、暂停、终止沙箱，管理模板和快照 | [控制面 REST](upstream/runtime/spec/openapi.yml) |
| 上传、下载文件，以及 envd HTTP 接口 | [envd HTTP](upstream/runtime/packages/envd/spec/envd.yaml) |
| 启动进程、接收输出、写入 stdin、连接进程和管理 PTY | [Process RPC](upstream/runtime/packages/envd/spec/process/process.proto) |
| 查询属性、操作目录、监听文件变化 | [Filesystem RPC](upstream/runtime/packages/envd/spec/filesystem/filesystem.proto) |

一次命令调用先通过控制面取得 `sandboxID`、域名和 envd 凭据，再访问该沙箱的 `/process.Process/Start`。进程结果通过事件流返回，生成的文件经 `GET /files` 下载。REST 路径版本与模板的 `envdVersion` 分别描述控制面接口和 guest 能力。控制面规范也包含管理端点，envd HTTP 中标记 `x-internal: true` 的操作供宿主机使用。

## 同一基线下的其他契约

| 范围 | 定义 |
| --- | --- |
| Edge 与 Hyperloop HTTP 服务 | [Edge](upstream/runtime/spec/openapi-edge.yml)、[Hyperloop](upstream/runtime/spec/openapi-hyperloop.yml) |
| 节点间调用及模板构建 | [Orchestrator、Info、Chunks、Template Manager、Volume](upstream/runtime/packages/orchestrator/) |
| 代理与控制面通信 | [Proxy RPC](upstream/runtime/packages/shared/pkg/grpc/proxy/proxy.proto) |
| 调度服务 | [Reservation 与 State](upstream/runtime/packages/scheduling-api/e2b/scheduling/v1/) |
| 旧版 guest 接口与升级交接 | [Legacy Process / Filesystem](upstream/runtime/packages/envd/internal/services/legacy/)、[Handover](upstream/runtime/packages/envd/spec/upgrade/handover.proto) |

## 核验与更新

`upstream/runtime/` 沿用上游路径。首次导入包含该提交的 4 份 OpenAPI 和全部 13 份 `.proto`。根目录的 `LICENSE` 是上游 Apache-2.0 许可证原文。[SHA256SUMS](SHA256SUMS) 记录这些原文件及许可证的字节摘要。在仓库根目录运行：

```sh
sha256sum -c SHA256SUMS
```

用 `protoc` 读取定义时，以 `packages/envd/spec` 为 guest 协议的导入根。节点与代理协议使用 `packages/orchestrator` 和 `packages/shared/pkg/grpc/proxy` 两个导入根。以上路径均相对于 `upstream/runtime/`。标准类型由 Protobuf 提供。调度协议另依赖 `buf/validate/validate.proto`，上游固定为 `buf.build/bufbuild/protovalidate:511051f7f4374c3ca873b53ae68a9288`。[上游生成指令](https://github.com/e2b-dev/runtime/blob/391b22443460fb30432da889b44aa413478d5717/packages/scheduling-api/generate.go#L3-L4)

更新上游基线时，一并更新提交号、来源链接、规范文件和摘要。SDK 路由、认证及运行时错误等跨接口行为可对照同一提交的[架构说明](https://github.com/e2b-dev/runtime/blob/391b22443460fb30432da889b44aa413478d5717/docs/ARCHITECTURE.md)。
