# E2BX 扩展

这里维护 E2BX 私有扩展。上游原始接口见[基线索引](../README.md)。

| 扩展 | 协议面 | 用途 |
| --- | --- | --- |
| [release-capture v1](release-capture/v1/README.md) · [OpenAPI](release-capture/v1/openapi.yml) | 控制面 REST | 释放沙箱时请求浮动快照，并按租户查询发布结果。 |
| [Computer Use v1](computer-use/v1/README.md) · [OpenAPI](computer-use/v1/openapi.yml) | guest envd HTTP / MCP | 已有 AgentHost 接入使用的桌面工具发现和调用协议。 |
| [Computer Use v2](computer-use/v2/README.md) · [OpenAPI](computer-use/v2/openapi.yml) | guest envd 原生 HTTP | `rust-envd` 本地验证过的原生工具接口。 |

新增能力时，可在 `extensions/<能力名>/v1/` 放一份 OpenAPI 或 `.proto`，并附最短的请求和响应示例。建议在说明中回答三个问题：

- 扩展面向控制面、guest agent，还是沙箱内的应用？
- 客户端如何知道服务端支持该能力？不支持时会收到什么结果？
- 这项扩展复用哪些基线类型、凭据和错误语义？新增部分如何识别？

可用 `e2bx.<能力名>.v1` 命名新的 Protobuf 包。示例路径与包名是起草约定，正式端点、消息和兼容承诺随各扩展定义。
