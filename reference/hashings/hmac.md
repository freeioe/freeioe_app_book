---

# HMAC消息认证码 (hashings.hmac)

HMAC（Hash-based Message Authentication Code）是一种基于密码哈希函数的消息认证码算法，用于验证数据完整性和真实性。

## 特性

- **用途**：消息认证，数据完整性验证
- **密钥支持**：使用密钥防止伪造
- **灵活性**：可与任何哈希函数结合使用
- **安全性**：比单纯的哈希更安全

## 工作原理

HMAC的计算公式：
```
HMAC(K, m) = H((K ⊕ opad) || H((K ⊕ ipad) || m))
```

其中：
- H：哈希函数（如SHA-256）
- K：密钥
- m：消息
- opad：外部填充（0x5C）
- ipad：内部填充（0x36）

## 使用方法

```lua
local hashings = require 'hashings'
local hmac = hashings.hmac
local sha256 = hashings.sha256

-- 创建HMAC-SHA256对象
local h = hmac:new(sha256, 'secret_key', 'message')
print(h:hexdigest())  -- 输出HMAC值

-- 流式处理
local h = hmac:new(sha256, 'secret_key')
h:update('part 1 ')
h:update('part 2')
print(h:hexdigest())

-- 使用不同哈希算法
local sha512 = hashings.sha512
local h = hmac:new(sha512, 'secret_key', 'message')
```

## 接口说明

### new

创建HMAC对象

#### 函数原型

```lua
function hmac:new(hash_module, key, data)
end
```

#### 参数说明

* hash_module
  哈希算法模块（如hashings.sha256、hashings.md5等）
* key
  密钥字符串
* data
  可选，初始消息数据

#### 返回值

返回HMAC对象

### update

更新HMAC计算

#### 函数原型

```lua
function hmac:update(data)
end
```

#### 参数说明

* data
  要添加到计算中的消息数据

### digest

获取二进制HMAC值

#### 函数原型

```lua
function hmac:digest()
end
```

#### 返回值

返回二进制字符串，长度取决于所用哈希算法

### hexdigest

获取十六进制HMAC值

#### 函数原型

```lua
function hmac:hexdigest()
end
```

#### 返回值

返回十六进制编码的字符串

### copy

复制HMAC对象及其状态

#### 函数原型

```lua
function hmac:copy()
end
```

#### 返回值

返回包含相同内部状态的新HMAC对象

## 使用示例

### API签名验证

```lua
local hashings = require 'hashings'
local hmac = hashings.hmac
local sha256 = hashings.sha256

-- 客户端：生成签名
local function sign_request(api_secret, method, path, body)
    local message = method .. path .. (body or '')
    local h = hmac:new(sha256, api_secret, message)
    return h:hexdigest()
end

-- 服务器：验证签名
local function verify_request(api_secret, signature, method, path, body)
    local message = method .. path .. (body or '')
    local h = hmac:new(sha256, api_secret, message)
    local calculated = h:hexdigest()
    return calculated == signature
end

-- 使用
local secret = 'my_api_secret_key'
local sig = sign_request(secret, 'GET', '/api/users', nil)
print('Signature:', sig)

local valid = verify_request(secret, sig, 'GET', '/api/users', nil)
print('Valid:', valid)
```

### JWT令牌签名

```lua
local hashings = require 'hashings'
local hmac = hashings.hmac
local sha256 = hashings.sha256

local function sign_jwt(header, payload, secret)
    local cjson = require 'cjson'

    -- Base64URL编码
    local function base64url_encode(data)
        local enc = data:gsub('+', '-'):gsub('/', '_'):gsub('=+$', '')
        return enc
    end

    local header_enc = base64url_encode(cjson.encode(header))
    local payload_enc = base64url_encode(cjson.encode(payload))
    local message = header_enc .. '.' .. payload_enc

    local h = hmac:new(sha256, secret, message)
    local signature = h:hexdigest()

    return message .. '.' .. signature
end

local token = sign_jwt(
    {alg='HS256', typ='JWT'},
    {sub='user123', exp=os.time()},
    'secret_key'
)
```

### Webhook验证

```lua
local hashings = require 'hashings'
local hmac = hashings.hmac
local sha256 = hashings.sha256

-- 服务器：发送webhook时添加签名
local function send_webhook(url, payload, secret)
    local http = require 'http.restful'
    local cjson = require 'cjson'

    local body = cjson.encode(payload)
    local h = hmac:new(sha256, secret, body)
    local signature = 'sha256=' .. h:hexdigest()

    return http:post(url, nil, body, nil, {
        ['X-Hub-Signature-256'] = signature
    })
end

-- 客户端：验证webhook签名
local function verify_webhook(payload, signature, secret)
    local signature_prefix = 'sha256='
    if not signature:find(signature_prefix, 1, true) then
        return false
    end

    local received_sig = signature:sub(#signature_prefix + 1)

    local h = hmac:new(sha256, secret, payload)
    local calculated_sig = h:hexdigest()

    return calculated_sig == received_sig
end
```

### 数据完整性保护

```lua
local hashings = require 'hashings'
local hmac = hashings.hmac
local sha256 = hashings.sha256

-- 存储数据时添加HMAC
local function store_with_auth(data, key)
    local h = hmac:new(sha256, key, data)
    local auth_tag = h:hexdigest()

    -- 存储格式: data.auth_tag
    return data .. '.' .. auth_tag
end

-- 读取并验证数据
local function load_and_verify(stored_data, key)
    local data, auth_tag = stored_data:match('^(.*)%.([A-F0-9]+)$')
    if not data then return nil end

    local h = hmac:new(sha256, key, data)
    local calculated_tag = h:hexdigest()

    if calculated_tag == auth_tag then
        return data, true
    else
        return nil, false  -- 验证失败
    end
end
```

## 安全建议

### 密钥管理

✅ **推荐做法**：
- 使用足够长度的密钥（至少与哈希输出长度相同）
- 密钥应该随机生成且保密
- 不同应用使用不同密钥
- 定期轮换密钥

❌ **避免**：
- 使用弱密钥或密码
- 在代码中硬编码密钥
- 共享密钥给不可信方

### 哈希算法选择

| 用途 | 推荐组合 |
| :--- | :--- |
| 通用安全 | HMAC-SHA256 |
| 高安全性 | HMAC-SHA512 |
| 兼容性 | HMAC-SHA1（不推荐新项目） |
| 非安全场景 | HMAC-MD5（不推荐） |

### 常见错误

❌ **错误1**：直接哈希密钥和消息
```lua
-- 不安全
local h = sha256:new(key .. message)
```

✅ **正确**：使用HMAC
```lua
-- 安全
local h = hmac:new(sha256, key, message)
```

❌ **错误2**：密钥过短
```lua
-- 不安全
local key = 'abc'  -- 太短
```

✅ **正确**：使用足够长的密钥
```lua
-- 安全
local key = '32_bytes_or_more_random_key_here'
```

## 性能特点

- **计算开销**：约为单纯哈希的两倍
- **内存占用**：比单纯哈希稍多
- **适用场景**：需要认证的场合

## 应用场景

### 1. API认证
- REST API签名
- 请求完整性验证
- 防止请求篡改

### 2. 令牌签名
- JWT令牌
- 会话令牌
- API密钥

### 3. 数据完整性
- 数据库记录保护
- 文件传输验证
- 消息队列确认

### 4. Webhook验证
- GitHub webhook
- Stripe webhook
- 自定义webhook

## 与其他认证码比较

| 类型 | 安全性 | 性能 | 密钥 |
| :--- | :--- | :--- | :--- |
| **HMAC** | **高** | **中等** | **对称** |
| 数字签名 | 最高 | 慢 | 非对称 |
| 简单哈希 | 低 | 快 | 无 |

选择HMAC的场景：
- 需要双方共享密钥
- 性能要求较高
- 不需要非对称加密的特性
