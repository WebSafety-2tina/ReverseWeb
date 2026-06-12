# 04 - JS 加密、签名与逆向

## 目标

还原前端生成 Token、签名、加密参数、设备指纹、CSRF、防重放字段的完整 pipeline，并给出可运行复现代码。

## 优先搜索关键词

```text
sign signature token nonce timestamp ts encrypt decrypt crypto digest sha sha1 sha256 md5 hmac aes rsa sm2 sm3 sm4
Authorization X-Sign X-Timestamp X-Nonce X-Token X-CSRF appKey secretKey publicKey privateKey
CryptoJS JSEncrypt forge SubtleCrypto window.crypto btoa atob encodeURIComponent JSON.stringify
fingerprint device visitor risk captcha challenge salt key iv mode padding
```

## 算法识别

- Hash：MD5、SHA1、SHA256、SHA512、SM3。
- MAC：HMAC-SHA256、HMAC-MD5。
- 对称加密：AES-CBC、AES-ECB、AES-GCM、DES、3DES、SM4。
- 非对称加密：RSA/PKCS1/OAEP、ECDH、SM2。
- 编码：Base64、Base64URL、Hex、URL encode、UTF-8、Latin1。
- 浏览器原生：WebCrypto `crypto.subtle.digest/importKey/sign/encrypt`。

## 签名 pipeline 检查

- 参与字段：method、path、query、body、timestamp、nonce、token、appKey、deviceId。
- 参数排序：ASCII、字典序、原始顺序、嵌套 JSON 是否递归排序。
- 空值规则：是否排除 `null`、空字符串、`undefined`、空数组。
- 编码顺序：先 stringify、先 URL encode、先 Base64、先压缩。
- Body hash：是否对 body 做 hash 后参与签名。
- 输出格式：hex/base64/base64url，大小写，截断长度。
- 防重放：timestamp 窗口、nonce 一次性、服务端下发 challenge。

## 混淆处理

- 字符串表和解码函数：定位 `_0x...`、array rotate、base64 decode。
- 控制流扁平化：先找网络调用附近的参数最终值，不急于完整反混淆。
- 动态执行：`eval`、`Function`、`setTimeout(string)`、WASM。
- source map：优先用 source map 定位原始函数名和 service 层。
- 断点策略：拦截 `fetch`、`XMLHttpRequest.send`、`axios.request`、`crypto.subtle`、`JSON.stringify`。

## 前端 Token 生成

分析 token 是否：

- 服务端签发：登录/初始化响应返回。
- 前端派生：由用户信息、时间戳、随机数、固定 secret 计算。
- 混合生成：服务端 challenge + 前端签名。
- 设备绑定：device_id/fingerprint 参与生成。
- 可预测：时间戳、递增 ID、弱随机、硬编码 salt。

## Python 复现模板

```python
import base64
import hashlib
import hmac
import json
import time
import uuid
from urllib.parse import urlencode

SECRET = b"replace-with-secret"

def stable_json(data):
    return json.dumps(data, separators=(",", ":"), ensure_ascii=False, sort_keys=True)

def body_hash(body):
    raw = stable_json(body).encode()
    return hashlib.sha256(raw).hexdigest()

def make_sign(method, path, query, body, token=""):
    ts = str(int(time.time() * 1000))
    nonce = uuid.uuid4().hex
    query_string = urlencode(sorted((query or {}).items()))
    msg = "\n".join([method.upper(), path, query_string, body_hash(body or {}), ts, nonce, token])
    sign = hmac.new(SECRET, msg.encode(), hashlib.sha256).hexdigest()
    return {"x-timestamp": ts, "x-nonce": nonce, "x-sign": sign}
```
