---

# HTTP Digest认证模块 (http.auth.digest)

本模块实现HTTP Digest认证，基于RFC 2617标准的摘要认证机制。

## 使用方法

```lua
local digest_auth = require 'http.auth.digest'

-- 创建认证对象
local auth = digest_auth:new('username', 'password', 'realm', 'nonce', 'auth')

-- 更新认证参数（通常从401响应中获取）
auth:set_realm('Protected Area')
auth:set_nonce('dcd98b7102dd2f0e8b11d0f600bfb0c093')

-- 生成认证头
local header_value = auth(nil, 'GET', '/api/data')
-- 返回: "Digest username=\"username\", realm=\"Protected Area\", ..."

-- 在RESTful客户端中使用
local restful = require 'http.restful'
local client = restful:new('https://api.example.com', 5000, nil, auth)
```

## 接口说明

### new

创建Digest认证对象

#### 函数原型

```lua
function digest_auth:new(user, passwd, realm, nonce, qop, algorithm, opaque)
end
```

#### 参数说明

* user
  用户名
* passwd
  密码
* realm
  认证域
* nonce
  服务器生成的随机数
* qop
  保护质量，可选值：'auth' 或 'auth-int'（可选）
* algorithm
  算法，默认'MD5'（可选）
* opaque
  服务器不透明数据（可选）

#### 返回值

返回Digest认证对象

### set_realm

设置认证域

#### 函数原型

```lua
function auth:set_realm(realm)
end
```

### set_nonce

设置nonce值

#### 函数原型

```lua
function auth:set_nonce(nonce)
end
```

### generate_nonce

生成新的nonce值

#### 函数原型

```lua
function auth:generate_nonce()
end
```

#### 返回值

返回基于时间戳生成的十六进制nonce字符串

### __call

生成认证头字符串

#### 函数原型

```lua
function auth:__call(headers, method, uri)
end
```

#### 参数说明

* headers
  请求头表（不使用）
* method
  HTTP方法（用于计算响应摘要）
* uri
  请求URI（用于计算响应摘要）

#### 返回值

返回Digest认证头字符串，包含：
- username
- realm
- nonce
- uri
- algorithm
- nc (nonce计数)
- cnonce (客户端nonce)
- response
- qop (如果设置)
- opaque (如果设置)

## 认证原理

Digest认证通过以下步骤实现更安全的身份验证：

1. **HA1计算**：`MD5(username:realm:password)`
2. **HA2计算**：`MD5(method:uri)`
3. **响应计算**：
   - 有qop：`MD5(HA1:nonce:nc:cnonce:qop:HA2)`
   - 无qop：`MD5(HA1:nonce:HA2)`

每次请求时，nc（nonce计数）会增加，确保每个请求的响应值唯一，防止重放攻击。

## 典型认证流程

```
客户端                    服务器
   |                        |
   |-------- GET ---------->|
   |                        |
   |<---- 401 Unauthorized --|
   |    WWW-Authenticate:   |
   |    Digest realm="...", |
   |    nonce="..."         |
   |                        |
   |-------- GET ---------->|
   |    Authorization:      |
   |    Digest username="...|
   |                        |
   |<---- 200 OK ----------|
```

## 安全特性

Digest认证相比Basic认证的安全性提升：

1. **密码不直接传输** - 只传输摘要值
2. **防重放攻击** - 使用nonce和nc计数
3. **防篡改** - 包含方法和URI的摘要
4. **时间敏感** - nonce通常有时间限制

## 注意事项

⚠️ **局限性**：
- MD5算法已被认为不够安全
- 需要保护realm和nonce等参数
- 仍然容易受到中间人攻击（除非使用HTTPS）

**建议**：
- 优先使用HTTPS + Bearer Token
- Digest主要用于兼容旧系统
