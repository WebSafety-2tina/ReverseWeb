# 01 - 总体逆向流程

## 总流程

1. 资产盘点：文件、页面、域名、Origin、接口、JS、source map、存储项、账号态。
2. 场景识别：登录、注册、AI 对话、上传、搜索、支付、后台管理、导出、审批等。
3. 请求时间线：按真实顺序还原入口页面、初始化、鉴权、业务请求、状态更新。
4. 依赖链：找出 Cookie、Token、nonce、timestamp、device_id、csrf、签名字段来源。
5. 前端调用链：从组件/hook/API client/拦截器追踪到具体请求。
6. 关键分支：权限判断、角色判断、状态判断、错误处理、重试、跳转。
7. 最小复现：只保留必要 Header、Cookie、Body、动态字段和前置请求。
8. 风险假设：未授权、越权、签名绕过、敏感信息泄露、XSS/CSRF/SSRF。
9. 验证闭环：改变一个变量，观察状态码、业务 code、数据归属和副作用。

## 资产盘点清单

- 入口 URL、最终 URL、Origin、Referer。
- 所有 XHR/fetch、GraphQL、SSE、WebSocket。
- 关键 JS：main、vendor、runtime、chunk、source map。
- API client：axios/fetch 封装、baseURL、interceptors、service 层。
- 存储：Cookie、localStorage、sessionStorage、IndexedDB、Cache Storage。
- 鉴权：Authorization、Set-Cookie、csrf、x-token、x-sign、refresh token。
- 数据对象：userId、orgId、tenantId、role、orderId、fileId、projectId、conversationId。

## 可验证假设格式

```text
假设：修改 orgId 可能触发水平越权。
证据：GET /api/projects?orgId=... 的 orgId 来自 localStorage，前端只做菜单隐藏。
验证：同一 token 替换 orgId，比较 200/403、返回数据归属和审计日志。
影响：若返回其他组织数据，则存在租户隔离失败。
```

## 链路化思维

- 信息泄露 → 枚举对象 ID → 未授权读取 → 越权修改 → 导出/下载/状态变更。
- 前端配置泄露 → 找到隐藏接口 → 低权限访问 → 权限字段篡改 → 后端未校验。
- 登录保持 → refresh 逻辑 → token 续期 → 会话固定/退出失效验证。
- 上传接口 → 预签名 URL → 文件类型绕过 → 回调确认 → 文件访问权限验证。
- 搜索/导出接口 → 条件放宽 → 批量数据访问 → 分页/游标枚举。
