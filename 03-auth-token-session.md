# 03 - Token、认证与登录保持

## 凭据类型

- Cookie Session：`sid`、`sessionid`、`JSESSIONID`、`connect.sid`。
- JWT：`access_token`、`id_token`、`refresh_token`。
- Bearer Token：`Authorization: Bearer ...`。
- 自定义 Header：`x-token`、`x-auth-token`、`x-api-key`、`x-session`。
- CSRF Token：Cookie + Header、meta 注入、初始化接口下发。
- 签名凭据：`x-sign`、`x-timestamp`、`x-nonce`、`appKey`。

## 获取入口

- 登录接口响应 body。
- `Set-Cookie`。
- 页面 HTML 注入：`window.__INITIAL_STATE__`、`__NEXT_DATA__`、meta tag。
- 初始化接口：`/api/init`、`/api/config`、`/session`、`/csrf`。
- 第三方回调：OAuth/OIDC code、state、PKCE。
- 匿名设备初始化：device_id、visitor_id、risk token。

## 登录保持分析

必须说明：

- access token 生命周期和过期表现。
- refresh token 存储位置、刷新接口、刷新条件。
- 401 后是否自动刷新并重放原请求。
- 多标签页同步：storage event、BroadcastChannel、SharedWorker。
- Remember me 是否改变 Cookie Max-Age 或 refresh token 生命周期。
- 退出登录是否服务端撤销 token，还是仅清前端存储。
- Cookie 属性：HttpOnly、Secure、SameSite、Domain、Path、Expires/Max-Age。

## JWT 检查

- Header：`alg`、`typ`、`kid`。
- Payload：`sub`、`iss`、`aud`、`exp`、`iat`、`nbf`、`scope`、`role`、`tenant`。
- 风险：过长有效期、payload 放敏感信息、未校验 aud/iss、弱密钥、kid 注入、alg none。

## 会话绑定

检查 token 是否绑定：

- IP、UA、设备 ID、tenantId、csrf、nonce、登录态版本号。
- 单点登录/单设备登录是否踢下线。
- 修改密码、退出登录、管理员禁用后 token 是否失效。

## 典型风险假设

```text
假设：退出登录只清理 localStorage，服务端 token 仍有效。
证据：logout 请求无 token revoke 响应，前端 removeItem 后跳转登录页。
验证：退出后复用旧 Authorization 调业务接口。
影响：本地登出不等于会话失效，泄露 token 可继续使用。
```
