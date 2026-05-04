---

# HTTP模块

本模块提供HTTP客户端功能，包括下载、RESTful API调用和多种认证方式。

## 模块列表

| 模块 | 说明 |
| :--- | :--- |
| [download](download.md) | HTTP下载模块，支持自动重定向 |
| [restful](restful.md) | RESTful API客户端模块 |
| [auth](auth/) | HTTP认证模块集合 |

## 使用示例

### HTTP下载

```lua
local http_download = require 'http.download'

-- 简单GET请求
local status, headers, body = http_download.get('https://example.com', '/api/data')
```

### RESTful API

```lua
local restful = require 'http.restful'

-- 创建RESTful客户端
local client = restful:new('https://api.example.com', 5000, {
    Accept = 'application/json'
})

-- GET请求
local status, body, headers = client:get('/api/users', { page = 1 })

-- POST请求
local status, body, headers = client:post('/api/users', nil, {
    name = 'John',
    email = 'john@example.com'
})
```

### 带认证的RESTful API

```lua
local restful = require 'http.restful'
local basic_auth = require 'http.auth.basic'

-- 使用Basic认证
local auth = basic_auth:new('username', 'password')
local client = restful:new('https://api.example.com', 5000, nil, auth)

local status, body, headers = client:get('/api/protected')
```
