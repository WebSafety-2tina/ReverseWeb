# 05 - 请求分析与复现

## HAR/Network 提取

优先提取：

- 文档入口、重定向链、最终 URL。
- XHR/fetch/GraphQL/SSE/WebSocket，先排除图片、字体、埋点、source map。
- 方法、URL、Query、Header、Cookie、Body、Content-Type。
- 响应 status、业务 code、Set-Cookie、CORS、安全头、Cache-Control。
- Initiator：由哪个 JS、函数或用户动作触发。
- Timing：blocked、dns、ssl、connect、send、wait、receive。

## 请求依赖链

```text
GET /
  -> Set-Cookie: sid
  -> HTML 注入 window.__CONFIG__.csrf
GET /api/init
  <- device_id, public_key
POST /api/login
  requires: csrf + device_id
  <- access_token, refresh_token
POST /api/action
  requires: Authorization + x-sign + timestamp + nonce
```

## 最小复现原则

- 不复制无关浏览器噪声 Header：`sec-ch-ua`、`sec-fetch-*` 通常先排除，除非风控依赖。
- 保留必要 Header：Authorization、Cookie、Origin、Referer、Content-Type、CSRF、签名字段。
- 使用 `requests.Session()` 保持 Cookie。
- 动态字段独立函数生成：timestamp、nonce、sign、device_id。
- 对关键响应做断言：状态码、业务 code、字段存在、数据归属。

## curl 转 requests 注意点

- `--compressed` 对应 requests 自动解压。
- `-b/-c` Cookie jar 对应 Session。
- `--data-raw` 默认 `application/x-www-form-urlencoded` 或原始字符串，需确认 Content-Type。
- JSON Body 使用 `json=payload`，表单用 `data=payload`。
- 文件上传用 `files=`，不要手写 multipart boundary。

## requests 骨架

```python
import requests

BASE_URL = "https://example.invalid"

s = requests.Session()
s.headers.update({
    "user-agent": "Mozilla/5.0",
    "accept": "application/json, text/plain, */*",
})

def init():
    r = s.get(BASE_URL + "/", timeout=20)
    r.raise_for_status()
    return r

def login(username, password):
    r = s.post(BASE_URL + "/api/login", json={"username": username, "password": password}, timeout=20)
    r.raise_for_status()
    data = r.json()
    if data.get("access_token"):
        s.headers["authorization"] = "Bearer " + data["access_token"]
    return data

def call(path, **kwargs):
    r = s.request(kwargs.pop("method", "GET"), BASE_URL + path, timeout=20, **kwargs)
    r.raise_for_status()
    return r.json()
```
