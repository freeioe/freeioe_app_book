---

# HTTP Basic认证模块 (http.auth.basic)

本模块实现HTTP Basic认证，基于用户名和密码的身份验证。

## 使用方法

```lua
local basic_auth = require 'http.auth.basic'

-- 创建认证对象
local auth = basic_auth:new('username', 'password')

-- 生成认证头
local header_value = auth(nil, 'GET', '/api/data')
-- 返回: "Basic dXNlcm5hbWU6cGFzc3dvcmQ="

-- 在RESTful客户端中使用
local restful = require 'http.restful'
local client = restful:new('https://api.example.com', 5000, nil, auth)
```

## 接口说明

### new

创建Basic认证对象

#### 函数原型

```lua
function basic_auth:new(user, passwd)
end
```

#### 参数说明

* user
  用户名
* passwd
  密码

#### 返回值

返回Basic认证对象

### __call

生成认证头字符串

#### 函数原型

```lua
function auth:__call(headers, method, uri)
end
```

#### 参数说明

* headers
  请求头表（本模块不使用）
* method
  HTTP方法（本模块不使用）
* uri
  请求URI（本模块不使用）

#### 返回值

返回格式为 `Basic <base64_encoded_credentials>` 的字符串

## 认证原理

Basic认证将用户名和密码按 `username:password` 格式组合，然后使用Base64编码，最后添加 `Basic ` 前缀。

```
Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==
```

## 安全注意事项

⚠️ **警告**：Basic认证将凭据以Base64编码传输，Base64是可逆编码，本质上等同于明文传输。

**建议**：
- 仅用于HTTPS连接
- 不要在不安全的HTTP连接上使用
- 定期更换密码
- 考虑使用更安全的认证方式（如Bearer Token或Digest）
