# 09 - SSE、WebSocket、GraphQL 与实时协议

## SSE

发现条件：

- `Content-Type: text/event-stream`
- `EventSource`
- `stream=true`
- AI 对话、通知、任务进度、日志流

分析内容：

- 端点、方法、Header、Body、鉴权。
- 事件字段：`event:`、`data:`、`id:`、`retry:`。
- 消息类型：delta、message、heartbeat、error、done、usage。
- 断线重连：Last-Event-ID、retry、客户端重试。
- 是否有 conversationId、messageId、taskId、model、temperature 等业务参数。

## WebSocket

分析内容：

- 握手 URL、Origin、Cookie、Authorization、Sec-WebSocket-Protocol。
- 首帧认证：token、subscribe、init、connection_init。
- 消息格式：JSON、MessagePack、Protobuf、纯文本、二进制。
- 状态机：connect → auth → subscribe → publish/receive → heartbeat → reconnect。
- 权限：是否能订阅他人房间、组织、订单、任务、聊天会话。

## GraphQL

分析内容：

- endpoint、operationName、query/mutation/subscription、variables。
- persisted query：hash、version、fallback。
- introspection 是否开启。
- 字段级权限：隐藏字段能否直接查询。
- 对象级权限：node ID、global ID、tenantId、ownerId。
- 批量请求：数组请求、多 operation。

## 实时协议风险假设

```text
假设：WebSocket 订阅 topic 未校验归属。
证据：客户端发送 {"type":"subscribe","topic":"order:123"}，topic 由前端拼接。
验证：登录普通账号订阅其他 orderId/topic。
安全结果：服务端返回 forbidden 或关闭连接。
风险结果：收到其他对象实时事件。
```

## Python 复现方向

- SSE：`requests.post(..., stream=True)` / `httpx.stream()`。
- WebSocket：`websockets.connect()` / `websocket-client`。
- GraphQL：普通 POST JSON，必要时复现 persisted query hash。
