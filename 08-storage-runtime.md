# 08 - 浏览器存储与运行态

## Cookie

检查字段：

- Domain、Path、Expires/Max-Age、HttpOnly、Secure、SameSite。
- 是否存 Token、Session、CSRF、device_id、tenantId、语言/区域。
- 是否跨子域共享，是否可被子域覆盖。
- SameSite=None 是否同时 Secure。

## localStorage / sessionStorage

重点查找：

```text
token access refresh jwt user profile permissions roles tenant org csrf device fingerprint config secret key
```

分类：

- 凭据类：Token、Session、CSRF、签名材料。
- 身份类：userId、role、permissions、tenantId、orgId。
- 状态类：当前项目、当前组织、流程步骤、草稿。
- 风控类：device_id、fingerprint、challenge、captcha token。
- 缓存类：用户资料、接口响应、feature flag。

## IndexedDB

关注：

- 离线消息、文件元数据、聊天记录、缓存响应。
- 大对象 blob、附件、导入导出数据。
- 加密数据库：密钥是否在 localStorage/sessionStorage。

## Service Worker / Cache Storage

检查：

- 是否缓存 API 响应或旧 bundle。
- 是否离线返回过期鉴权状态。
- 是否能从 cache 找到历史接口响应、配置或 source map。
- fetch handler 是否改写请求、加 Header、走代理。

## 前端运行态

定位：

- `window.__INITIAL_STATE__`、`__NEXT_DATA__`、`__NUXT__`、`__APP_CONFIG__`。
- Redux/Vuex/Pinia/Zustand store。
- React Query/SWR cache。
- 全局 axios 实例、GraphQL client、WebSocket 实例。

## 登录保持相关运行态

- token 是否只在内存，刷新页面后如何恢复。
- 多标签页是否同步登录/退出。
- refresh 是否由定时器、拦截器或页面可见性事件触发。
- 本地时间偏移是否影响 token 过期判断。

## 输出模板

```markdown
| 存储位置 | key/name | 类型 | 来源 | 用途 | 风险/依赖 |
|---|---|---|---|---|---|
| localStorage | access_token | 凭据 | POST /api/login | Authorization | XSS 后可窃取 |
```
