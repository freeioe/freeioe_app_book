---

# PBKDF2密钥派生函数 (hashings.pbkdf2)

PBKDF2（Password-Based Key Derivation Function 2）是一种基于密码的密钥派生函数，通过迭代哈希运算将密码转换为安全的密钥。

## 特性

- **用途**：密码存储、密钥派生
- **可配置**：可调整迭代次数和盐值
- **抗暴力破解**：通过增加迭代次数提高破解成本
- **标准算法**：PKCS #5 RFC 2898标准

⚠️ **重要**：PBKDF2的正确使用需要足够的迭代次数。现代推荐至少100,000次迭代（2024年标准）。

## 使用方法

```lua
local hashings = require 'hashings'
local pbkdf2 = hashings.pbkdf2
local sha256 = hashings.sha256

-- 派生密钥
local password = 'user_password'
local salt = 'random_salt_value'
local iterations = 10000  -- 应根据当前硬件性能调整

local derived_key = pbkdf2:pbkdf2(sha256, password, salt, iterations)
print(derived_key)  -- 输出64字符十六进制字符串（SHA-256）
```

## 接口说明

### pbkdf2

派生密钥

#### 函数原型

```lua
function pbkdf2:pbkdf2(hash_module, password, salt, iterations)
end
```

#### 参数说明

* hash_module
  哈希算法模块（推荐sha256或sha512）
* password
  用户密码字符串
* salt
  盐值字符串（应该是随机且唯一的）
* iterations
  迭代次数，越大越安全但越慢

#### 返回值

返回十六进制编码的派生密钥字符串

## 使用示例

### 密码哈希存储

```lua
local hashings = require 'hashings'
local pbkdf2 = hashings.pbkdf2
local sha256 = hashings.sha256

-- 生成随机盐
local function generate_salt()
    local chars = 'abcdefghijklmnopqrstuvwxyz0123456789'
    local salt = {}
    for i = 1, 32 do
        local rand = math.random(1, #chars)
        table.insert(salt, chars:sub(rand, rand))
    end
    return table.concat(salt)
end

-- 哈希密码
local function hash_password(password)
    local salt = generate_salt()
    local iterations = 100000  -- 2024年推荐标准

    local derived_key = pbkdf2:pbkdf2(sha256, password, salt, iterations)

    -- 存储格式: iterations$salt$derived_key
    return string.format('%d$%s$%s', iterations, salt, derived_key)
end

-- 验证密码
local function verify_password(stored_hash, password)
    local iterations, salt, derived_key = stored_hash:match('^(%d+)$([^$]+)$([A-F0-9]+)$')
    if not iterations then return false end

    iterations = tonumber(iterations)
    local check_key = pbkdf2:pbkdf2(sha256, password, salt, iterations)

    return check_key == derived_key
end

-- 使用
local stored = hash_password('my_secure_password')
print('Stored:', stored)

local valid = verify_password(stored, 'my_secure_password')
print('Valid:', valid)  -- true

local invalid = verify_password(stored, 'wrong_password')
print('Invalid:', invalid)  -- false
```

### 密钥派生用于加密

```lua
local hashings = require 'hashings'
local pbkdf2 = hashings.pbkdf2
local sha256 = hashings.sha256

-- 从密码派生加密密钥和IV
local function derive_encryption_keys(password, salt)
    local iterations = 100000

    -- 派生主密钥
    local main_key = pbkdf2:pbkdf2(sha256, password, salt, iterations)

    -- 分割为密钥和IV（假设使用AES-256）
    local key = main_key:sub(1, 64)  -- 32字节 = 64个十六进制字符
    local iv = main_key:sub(65, 80)   -- 8字节用于IV

    return key, iv
end

local key, iv = derive_encryption_keys('encryption_password', 'unique_salt')
print('Key:', key)
print('IV:', iv)
```

### API密钥派生

```lua
local hashings = require 'hashings'
local pbkdf2 = hashings.pbkdf2
local sha256 = hashings.sha256

-- 从主密钥派生多个子密钥
local function derive_api_key(master_key, service_name, salt)
    local iterations = 50000
    local combined = service_name .. salt

    local api_key = pbkdf2:pbkdf2(sha256, master_key, combined, iterations)

    -- 截取为所需长度（例如32字符）
    return api_key:sub(1, 32)
end

local master = 'super_secret_master_key'
local key1 = derive_api_key(master, 'service1', 'salt1')
local key2 = derive_api_key(master, 'service2', 'salt2')
```

### 令牌生成

```lua
local hashings = require 'hashings'
local pbkdf2 = hashings.pbkdf2
local sha512 = hashings.sha512

-- 生成安全令牌
local function generate_token(user_id, secret, valid_for)
    local salt = os.time() .. user_id  -- 时间戳作为盐
    local iterations = 10000

    local token = pbkdf2:pbkdf2(sha512, secret, salt, iterations)

    -- 令牌包含用户ID和过期时间信息
    return string.format('%s:%d:%s', user_id, os.time() + valid_for, token:sub(1, 32))
end

local token = generate_token('user123', 'app_secret', 3600)  -- 1小时有效期
print('Token:', token)
```

## 安全建议

### 迭代次数选择

| 年份 | 推荐迭代次数 | 说明 |
| :--- | :--- | :--- |
| 2020 | 10,000 | 最低标准 |
| 2024 | 100,000 | 推荐标准 |
| 2026+ | 200,000+ | 随硬件性能提升 |

选择原则：
- 在可接受的延迟范围内尽可能高
- 目标：每次验证耗时100-500ms
- 定期评估和调整

### 盐值管理

✅ **推荐做法**：
```lua
-- 使用加密安全的随机数生成器
local function generate_salt()
    -- 16字节（128位）随机盐
    local salt = {}
    for i = 1, 16 do
        salt[i] = string.char(math.random(0, 255))
    end
    return table.concat(salt)
end

-- 每个密码使用唯一盐
local salt = generate_salt()
```

❌ **避免**：
```lua
-- 不要使用固定盐
local salt = 'static_salt'  -- 不安全

-- 不要使用可预测信息作为盐
local salt = user_id  -- 可预测
```

### 哈希算法选择

| 算法 | 推荐度 | 说明 |
| :--- | :--- | :--- |
| SHA-256 | ✅ 推荐 | 性能与安全性平衡 |
| SHA-512 | ✅ 推荐 | 更高安全性 |
| SHA-1 | ❌ 不推荐 | 已过时且不安全 |
| MD5 | ❌ 禁止 | 不安全 |

## 迁移指南

### 从简单哈希迁移

❌ **旧代码（不安全）**：
```lua
local md5 = require 'hashings.md5'

local function hash_password(password)
    local h = md5:new(password)
    return h:hexdigest()
end
```

✅ **新代码（安全）**：
```lua
local hashings = require 'hashings'

local function hash_password(password)
    local salt = generate_salt()
    local iterations = 100000
    local derived = pbkdf2:pbkdf2(sha256, password, salt, iterations)
    return string.format('%d$%s$%s', iterations, salt, derived)
end
```

### 渐进式迁移

```lua
-- 混合验证：支持旧哈希和新PBKDF2
local function verify_password(stored, password)
    if stored:find('$') then
        -- 新格式：PBKDF2
        return verify_pbkdf2(stored, password)
    else
        -- 旧格式：简单哈希，验证后更新
        local valid = verify_legacy(stored, password)
        if valid then
            -- 更新为新格式
            return true, 'needs_rehash'
        end
        return false
    end
end
```

## 性能优化

### 并行验证

```lua
-- 批量验证时使用协程
local function verify_batch(credentials_list)
    local threads = {}
    for i, cred in ipairs(credentials_list) do
        threads[i] = coroutine.create(function()
            return verify_password(cred.stored, cred.password)
        end)
    end

    local results = {}
    for i, thread in ipairs(threads) do
        local success, valid = coroutine.resume(thread)
        results[i] = valid
    end

    return results
end
```

### 迭代次数调整

```lua
-- 根据硬件自动调整迭代次数
local function calibrate_iterations(target_time_ms)
    local iterations = 10000
    local start = os.clock()

    pbkdf2:pbkdf2(sha256, 'test', 'salt', iterations)
    local elapsed = (os.clock() - start) * 1000

    -- 调整到目标时间
    iterations = math.floor(iterations * target_time_ms / elapsed)
    return iterations
end

local optimal_iterations = calibrate_iterations(200)  -- 目标200ms
```

## 最佳实践

### 1. 密码存储

```lua
-- 推荐的密码哈希配置
local config = {
    algorithm = sha256,
    iterations = 100000,
    salt_length = 16,
    output_length = 32
}
```

### 2. 密钥派生

```lua
-- 加密密钥派生
local function derive_key(password, salt, key_length)
    local iterations = 100000

    local derived = pbkdf2:pbkdf2(sha256, password, salt, iterations)

    -- 截取所需长度
    return derived:sub(1, key_length * 2)  -- 十六进制字符数 = 字节数 * 2
end
```

### 3. 错误处理

```lua
local function safe_pbkdf2(hash_fn, password, salt, iterations)
    if not password or not salt or not iterations then
        return nil, 'Missing required parameters'
    end

    if iterations < 10000 then
        return nil, 'Iterations too low (minimum 10000)'
    end

    local ok, result = pcall(pbkdf2.pbkdf2, pbkdf2, hash_fn, password, salt, iterations)
    if not ok then
        return nil, 'PBKDF2 computation failed'
    end

    return result
end
```

## 注意事项

⚠️ **重要**：

1. **迭代次数**：必须足够高以抵抗暴力破解
2. **盐值唯一性**：每个密码/密钥必须使用唯一盐
3. **盐值随机性**：使用加密安全的随机数生成器
4. **性能权衡**：平衡安全性和用户体验
5. **定期审查**：随着硬件发展调整迭代次数

## 替代方案

虽然PBKDF2是标准选择，但也可以考虑：

- **bcrypt**：专为密码设计，内置盐值，可调整成本因子
- **scrypt**：抗GPU/ASIC攻击，内存密集型
- **Argon2**：2015年密码哈希竞赛 winner，抗侧信道攻击

选择建议：
- 通用用途：PBKDF2
- 密码存储：bcrypt或Argon2
- 高安全性：scrypt或Argon2
