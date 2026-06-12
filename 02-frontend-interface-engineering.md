# 02 - 前端接口工程师级分析

## 分析目标

不仅看 Network 面板，还要还原接口在前端工程里的真实生命周期：从 UI 事件到请求构造、拦截器注入、响应处理、状态落点和权限展示。

## 调用链定位

优先定位：

- 组件：button submit、form onFinish、useEffect、route loader、server action。
- hook：`useLogin`、`useUser`、`useQuery`、`useMutation`、自定义业务 hook。
- service/API client：`api.ts`、`request.ts`、`http.ts`、`services/*`。
- 拦截器：axios request/response interceptors、fetch wrapper、GraphQL link。
- 状态管理：Redux、Vuex、Pinia、Zustand、MobX、React Query、SWR。
- 路由守卫：beforeEach、middleware、loader、layout auth gate。

## 请求构造检查

- baseURL 来源：env、runtime config、window 注入、meta、localStorage、proxy。
- path 拼接：是否可控、是否存在路径穿越或多租户路径参数。
- Query 序列化：数组格式、空值处理、排序、编码。
- Body 序列化：JSON、FormData、URLSearchParams、protobuf、加密 blob。
- Header：Authorization、x-token、x-sign、x-csrf-token、x-tenant-id、x-device-id。
- Credentials：fetch `credentials`、axios `withCredentials`、跨域 Cookie。

## 响应处理检查

- HTTP status 与业务 code 是否双层判断。
- 401/403 是否触发刷新 token、跳登录或静默重试。
- 业务错误是否被统一 toast 掩盖。
- 响应数据写入哪里：store、cache、localStorage、IndexedDB、内存变量。
- 敏感字段是否被前端持久化：role、permissions、token、secret、profile、PII。

## 前端权限分析

- 菜单权限：是否只隐藏 UI，接口是否仍可访问。
- 按钮权限：是否只在前端判断 role/permission。
- 路由权限：直接访问 URL 是否触发后端验证。
- 字段权限：前端是否返回了不该展示的字段后再过滤。
- 租户权限：orgId/tenantId 是否由前端传入且可篡改。

## 输出表

```markdown
| 接口 | 前端调用点 | 请求构造 | 鉴权/签名注入点 | 响应落点 | 可测风险 |
|---|---|---|---|---|---|
| POST /api/order | src/services/order.ts:createOrder | JSON body | request interceptor 注入 token/sign | orderStore.current | userId/orgId 可篡改 |
```
