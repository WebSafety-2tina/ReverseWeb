# 07 - XSS、CSRF、SSRF 与边界风险

## XSS 分析

### 输入点

- 搜索框、昵称、简介、评论、富文本、文件名、导入内容、URL 参数。
- Markdown、HTML 预览、AI 对话渲染、日志查看、报表字段。
- 管理后台审核页、客服工单、通知模板。

### 输出点

- `innerHTML`、`dangerouslySetInnerHTML`、Vue `v-html`、Svelte `{@html}`。
- Markdown renderer、highlight.js、DOMPurify 配置。
- URL 跳转、iframe src、srcdoc、SVG、data URL。

### 判断点

- 是否上下文相关编码：HTML、属性、URL、JS 字符串、CSS。
- 是否允许危险标签/属性：script、iframe、object、on*、style、href=javascript:。
- 是否存在存储型路径：提交后由管理员或其他用户查看。
- CSP 是否限制 inline script、外部源、frame。

## CSRF 分析

检查：

- 认证是否依赖 Cookie 自动携带。
- 关键写操作是否要求 CSRF Token。
- Token 是否与 Session 绑定、是否一次性或可复用。
- Cookie SameSite：Strict/Lax/None。
- CORS 是否允许跨站带凭据。
- Content-Type 是否可由普通表单发起。

高风险接口：

- 修改密码、绑定邮箱/手机、转账/支付、创建订单、删除资源、角色变更、Webhook 配置。

## SSRF 分析

前端常见入口：

- URL 预览、图片抓取、头像上传 URL、导入远程文件。
- Webhook、回调地址、RSS/站点地图、PDF/截图生成。
- 对象存储迁移、在线文档转换、AI 插件/工具调用。

请求证据：

- 接口参数含 `url`、`callback`、`webhook`、`target`、`redirect`、`imageUrl`、`fetchUrl`。
- 响应包含远程内容、截图、标题、状态码或错误信息。
- 后端错误泄露 DNS、连接超时、内网 IP、协议限制。

验证思路：

- 先用可控普通 URL 验证服务端是否发起请求。
- 再测试协议限制、重定向、内网网段、DNS rebinding 防护。
- 线下 CTF 中优先使用比赛提供的内网 DNS/回显服务/日志服务作为证据。

## CORS 与安全头

检查：

- `Access-Control-Allow-Origin: *` 是否与 credentials 同用。
- 是否反射 Origin。
- `Access-Control-Allow-Credentials: true`。
- CSP、X-Frame-Options/frame-ancestors、X-Content-Type-Options、Referrer-Policy。

## 输出要求

不要只说“可能有 XSS/CSRF/SSRF”。必须给：输入点、输出点/服务端请求点、当前防护、验证请求、预期安全结果、风险结果。
