---

# RESTful API模块 (http.restful)

本模块提供RESTful API客户端功能，支持常见的HTTP方法和多种认证方式。

## 特性

- 支持GET、POST、PUT、DELETE方法
- 支持JSON数据自动序列化
- 支持自定义请求头
- 支持多种认证方式（Basic、Bearer、Digest）
- 可配置超时时间
- 自动URL参数编码

## 使用方法

```lua
local restful = require 'http.restful'

-- 创建RESTful客户端
local client = restful:new('https://api.example.com', 5000)

-- GET请求
local status, body, resp_headers = client:get('/api/users', { page = 1, limit = 10 })

-- POST JSON数据
local status, body, resp_headers = client:post('/api/users', nil, {
    name = 'John Doe',
    email = 'john@example.com'
})

-- PUT请求
local status, body, resp_headers = client:put('/api/users/1', nil, {
    name = 'Jane Doe'
})

-- DELETE请求
local status, body, resp_headers = client:delete('/api/users/1')
```

### 使用认证

```lua
local restful = require 'http.restful'
local basic_auth = require 'http.auth.basic'

-- 使用Basic认证（简写形式）
local client = restful:new('https://api.example.com', 5000, nil, {'username', 'password'})

-- 使用Basic认证（完整形式）
local auth = basic_auth:new('username', 'password')
local client = restful:new('https://api.example.com', 5000, nil, auth)
```

## 接口说明

### new

创建RESTful客户端实例

#### 函数原型

```lua
function restful:new(host, timeout, headers, auth)
end
```

#### 参数说明

* host
  目标主机地址，格式如 `https://api.example.com`
* timeout
  可选，请求超时时间，单位毫秒（默认5000）
* headers
  可选，默认请求头表（默认包含`Accept: application/json`）
* auth
  可选，认证对象或`{user, password}`表

#### 返回值

返回RESTful客户端对象

### request

执行HTTP请求（通用方法）

#### 函数原型

```lua
function client:request(method, url, params, data, content_type)
end
```

#### 参数说明

* method
  HTTP方法：'GET'、'POST'、'PUT'、'DELETE'
* url
  请求路径
* params
  可选，URL查询参数表
* data
  可选，请求体数据（表类型会自动编码为JSON）
* content_type
  可选，Content-Type（默认`text/plain`，表类型数据默认为`application/json`）

#### 返回值

成功返回：
1. status - HTTP状态码
2. body - 响应体
3. headers - 响应头表

失败返回：
1. nil
2. error - 错误信息

### get

执行GET请求

#### 函数原型

```lua
function client:get(url, params, data, content_type)
end
```

### post

执行POST请求

#### 函数原型

```lua
function client:post(url, params, data, content_type)
end
```

### put

执行PUT请求

#### 函数原型

```lua
function client:put(url, params, data, content_type)
end
```

### delete

执行DELETE请求

#### 函数原型

```lua
function client:delete(url, params, data, content_type)
end
```

## 认证支持

支持三种认证方式，详见[auth目录](auth/)：

1. **Basic认证** - HTTP基本认证
   ```lua
   local basic_auth = require 'http.auth.basic'
   local auth = basic_auth:new('username', 'password')
   ```

2. **Bearer认证** - Token认证
   ```lua
   local bearer_auth = require 'http.auth.bearer'
   local auth = bearer_auth:new('token_string')
   ```

3. **Digest认证** - HTTP摘要认证
   ```lua
   local digest_auth = require 'http.auth.digest'
   local auth = digest_auth:new('username', 'password', realm, nonce, qop)
   ```

## URL参数编码

所有查询参数会自动进行URL编码，支持任意字符。
