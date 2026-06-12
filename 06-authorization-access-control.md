# 06 - 未授权、越权与访问控制

## 分析目标

判断后端是否真正校验身份、角色、租户、对象归属和状态，而不是只依赖前端隐藏按钮或路由守卫。

## 未授权访问

检查方式：

- 去掉 Authorization / Cookie 后访问接口。
- 使用过期 token、伪造 token、空 token。
- 只保留匿名初始化 Cookie。
- 直接访问前端未暴露但 bundle 中存在的接口。

观察：

- HTTP status：200/401/403/302。
- 业务 code：是否返回成功但数据为空。
- 响应字段：是否泄露用户、配置、权限、对象详情。
- 副作用：POST/PUT/DELETE 是否真的执行。

## 水平越权 / IDOR

重点参数：

```text
userId uid accountId orgId tenantId teamId projectId orderId fileId invoiceId conversationId messageId roleId permissionId
```

验证方式：

- 同一账号替换对象 ID。
- A 账号创建对象，B 账号访问/修改/删除。
- 替换 tenantId/orgId，观察数据归属。
- 改分页、过滤条件、导出条件扩大范围。

## 垂直越权

检查：

- 普通用户调用管理员接口。
- 修改 role、permission、isAdmin、scope 字段。
- 前端菜单隐藏但接口可直接请求。
- 管理端 API 与用户端 API 共用 token。
- GraphQL 查询隐藏字段或 mutation。

## 多租户隔离

关注：

- tenantId/orgId 是否由前端传入。
- token 内 tenant 是否与请求参数绑定。
- 切换组织时是否只改 localStorage。
- 导出、搜索、批量接口是否漏租户条件。

## 状态越权

检查业务状态机：

- 未支付订单是否能发货。
- 未审核对象是否能发布。
- 已删除/已归档对象是否能恢复或访问。
- 非拥有者是否能取消、审批、转移、导出。

## 输出模板

```markdown
### 越权假设：替换 `fileId` 可读取他人文件

- 证据：`GET /api/files/{fileId}` 由前端直接拼接 fileId，响应包含下载 URL。
- 前置状态：登录普通用户。
- 验证请求：使用同一 token 替换为其他用户 fileId。
- 预期安全结果：403 或 404。
- 风险结果：200 且返回其他用户文件元数据/下载 URL。
```
