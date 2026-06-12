# 前端总逆向分析：模块调用索引

本文件是总调度入口。根据用户目标选择一个主模块，必要时叠加多个副模块。

## 模块列表

1. `00-scope-and-evidence.md` — CTF/线下攻防沙盒作用域、证据优先级、工作原则。
2. `01-overall-workflow.md` — 总体逆向流程、资产盘点、时间线、依赖链、复现闭环。
3. `02-frontend-interface-engineering.md` — 前端接口工程师级分析：组件、hook、API client、拦截器、状态管理、缓存。
4. `03-auth-token-session.md` — Token、Cookie、JWT、登录保持、刷新机制、会话绑定和退出登录。
5. `04-js-crypto-reverse.md` — JS 加密、签名、哈希、密钥、混淆、参数还原和 Python 复现。
6. `05-request-analysis-replay.md` — HTTP 请求分析、HAR 提取、curl/requests 复现、依赖字段追踪。
7. `06-authorization-access-control.md` — 未授权、越权访问、IDOR、租户隔离、角色权限、前后端信任边界。
8. `07-xss-csrf-ssrf.md` — XSS、CSRF、SSRF、CORS、安全头和输入输出边界。
9. `08-storage-runtime.md` — Cookie/localStorage/sessionStorage/IndexedDB/Service Worker/前端运行态分析。
10. `09-stream-graphql-realtime.md` — SSE、WebSocket、GraphQL、流式 AI 对话和实时协议。
11. `10-report-and-templates.md` — 输出报告模板、风险分级、接口表、复现代码模板、质量自检。

## 调用规则

- 用户说“整体分析 / 自动识别 / 看看这个站”：调用 `00` + `01` + `02` + `05` + `10`。
- 用户说“接口逆向 / API / 请求复现 / HAR / curl”：调用 `01` + `05` + `10`。
- 用户说“前端源码 / bundle / sourcemap / axios / fetch”：调用 `02` + `03` + `04` + `08`。
- 用户说“加密 / 签名 / token 参数 / x-sign / JS 逆向”：调用 `04` + `05` + `10`。
- 用户说“登录 / 保持登录 / Cookie / JWT / refresh token”：调用 `03` + `08` + `05`。
- 用户说“未授权 / 越权 / IDOR / 权限绕过”：调用 `06` + `03` + `05` + `10`。
- 用户说“XSS / CSRF / SSRF / CORS / 安全头”：调用 `07` + `05` + `10`。
- 用户说“SSE / WebSocket / GraphQL / 流式对话”：调用 `09` + `03` + `05`。
- 用户说“比赛 / 线下 CTF / 局域网 DNS / 仿真域名”：调用 `00` 并贯穿所有模块。

## 输出习惯

推荐输出顺序：

1. 结论：当前识别出的业务场景、关键风险或可复现程度。
2. 关键证据：请求、响应、JS 位置、存储项、状态变化。
3. 链路：从入口到关键接口/数据/业务目标的顺序。
4. 可验证假设：风险点、证据、验证方式、预期结果。
5. 复现代码或请求模板。
6. 下一步最小验证动作。
