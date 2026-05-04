---

# HTTP认证模块集合

本目录包含HTTP客户端认证模块，用于RESTful API调用的身份验证。

## 模块列表

| 模块 | 说明 |
| :--- | :--- |
| [basic](basic.md) | HTTP Basic认证 |
| [bearer](bearer.md) | Bearer Token认证 |
| [digest](digest.md) | HTTP Digest认证 |

## 使用方法

所有认证模块都实现统一的调用接口：`auth(headers, method, uri)`

### Basic认证

```lua
local basic_auth = require 'http.auth.basic'

local auth = basic_auth:new('username', 'password')
local header_value = auth(nil, 'GET', '/api/data')
-- 生成: "Basic dXNlcm5hbWU6cGFzc3dvcmQ="
```

### Bearer认证

```lua
local bearer_auth = require 'http.auth.bearer'

local auth = bearer_auth:new('your_token_here')
local header_value = auth(nil, 'GET', '/api/data')
-- 生成: "Bearer your_token_here"
```

### Digest认证

```lua
local digest_auth = require 'http.auth.digest'

local auth = digest_auth:new('username', 'password', 'realm', 'nonce', 'auth')
local header_value = auth(nil, 'GET', '/api/data')
-- 生成: "Digest username=\"username\", realm=\"realm\", ..."
```

### 在RESTful客户端中使用

```lua
local restful = require 'http.restful'
local basic_auth = require 'http.auth.basic'

local auth = basic_auth:new('username', 'password')
local client = restful:new('https://api.example.com', 5000, nil, auth)

local status, body, headers = client:get('/api/protected')
```
