---

# Bearer Token认证模块 (http.auth.bearer)

本模块实现Bearer Token认证，常用于OAuth 2.0和现代API认证。

## 使用方法

```lua
local bearer_auth = require 'http.auth.bearer'

-- 创建认证对象
local auth = bearer_auth:new('your_access_token_here')

-- 生成认证头
local header_value = auth(nil, 'GET', '/api/data')
-- 返回: "Bearer your_access_token_here"

-- 在RESTful客户端中使用
local restful = require 'http.restful'
local client = restful:new('https://api.example.com', 5000, nil, auth)

local status, body, headers = client:get('/api/protected')
```

## 接口说明

### new

创建Bearer认证对象

#### 函数原型

```lua
function bearer_auth:new(token)
end
```

#### 参数说明

* token
  访问令牌字符串

#### 返回值

返回Bearer认证对象

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

返回格式为 `Bearer <token>` 的字符串

## 认证原理

Bearer认证是一种简单的令牌认证机制，客户端在每次请求时在Authorization头中携带访问令牌。

```
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

## 使用场景

Bearer Token认证通常用于：

1. **OAuth 2.0** - 标准的授权框架
2. **JWT (JSON Web Tokens)** - 自包含的令牌
3. **API密钥** - 简单的API访问控制

## 安全注意事项

**建议**：
- 仅用于HTTPS连接
- 妥善保管令牌，不要泄露
- 设置合理的令牌过期时间
- 使用刷新令牌机制更新过期的访问令牌
- 实施令牌撤销机制

## 与其他认证方式的比较

| 认证方式 | 安全性 | 复杂度 | 适用场景 |
| :--- | :--- | :--- | :--- |
| Bearer | 高（HTTPS） | 低 | 现代API、OAuth 2.0 |
| Basic | 低（HTTPS） | 低 | 简单场景 |
| Digest | 中 | 中 | 旧系统兼容 |
