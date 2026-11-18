# Release capture v1

Slot Pool 生命周期网关支持在释放沙箱时请求浮动快照。请求沿用控制面的 `X-API-Key`，网关用它确定租户。客户端应从部署配置确认目标网关支持此扩展。`DELETE` 会释放沙箱，不能用作无副作用的能力探测。[OpenAPI 定义](openapi.yml)

以下请求使用同一网关和租户。`sandbox-123` 代表待释放的沙箱。

```http
DELETE /sandboxes/sandbox-123 HTTP/1.1
X-API-Key: <tenant-key>
X-Sandbox-Capture: floating
```

```http
HTTP/1.1 202 Accepted
Content-Type: application/json

{"capture_id":"0123456789abcdef0123456789abcdef","state_url":"/captures/0123456789abcdef0123456789abcdef"}
```

收到 `202` 后，沙箱已逻辑删除。同一租户重试相同沙箱的 `floating` 请求会得到同一份受理结果。使用返回的 `state_url` 查询快照是否已发布。

```http
GET /captures/0123456789abcdef0123456789abcdef HTTP/1.1
X-API-Key: <tenant-key>
```

```http
HTTP/1.1 200 OK
Content-Type: application/json

{"capture_id":"0123456789abcdef0123456789abcdef","state":"committed","seq":1}
```

`in_progress` 表示稍后再查。`committed` 给出已发布快照的 `seq`，从该快照恢复属于独立能力。`failed` 给出失败原因，沙箱仍保持逻辑删除。

`DELETE /v2/sandboxes/{sandboxID}` 同样支持此策略。省略策略头、值为空或传 `none` 时，普通释放成功返回 `204`。网关禁用捕获时返回 `400 capture unavailable`，当前绑定不支持捕获时返回 `409 capture not possible`。这两种情况下沙箱均保留。
