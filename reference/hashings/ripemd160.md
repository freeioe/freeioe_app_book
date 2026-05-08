---

# RIPEMD-160哈希算法 (hashings.ripemd160)

RIPEMD-160（RACE Integrity Primitives Evaluation Message Digest 160）是产生160位哈希值的加密哈希函数。作为MD5和SHA-1的替代方案，在比特币等密码学货币中被广泛使用。

## 特性

- **摘要长度**：160位（20字节）
- **块大小**：64字节（512位）
- **输出格式**：通常表示为40个十六进制字符
- **安全性**：中等，比MD5和SHA-1更安全
- **用途**：密码学货币、数字签名、数据完整性

## 与其他哈希算法比较

| 算法 | 摘要长度 | 速度 | 安全性 | 状态 |
| :--- | :--- | :--- | :--- | :--- |
| MD5 | 128位 | 快 | 已破解 | 不推荐 |
| SHA-1 | 160位 | 快 | 已破解 | 不推荐 |
| **RIPEMD-160** | **160位** | **中等** | **尚可** | **可用** |
| SHA-256 | 256位 | 中等 | 高 | 推荐 |

## 使用方法

```lua
local ripemd160 = require 'hashings.ripemd160'

-- 创建RIPEMD-160哈希对象并计算哈希
local h = ripemd160:new('Hello World')
print(h:hexdigest())  -- 输出: 40位十六进制字符串

-- 流式处理
local h = ripemd160:new()
h:update('First part ')
h:update('Second part')
print(h:hexdigest())
```

## 接口说明

### new

创建RIPEMD-160哈希对象

#### 函数原型

```lua
function ripemd160:new(data)
end
```

#### 参数说明

* data
  可选，初始数据字符串

#### 返回值

返回RIPEMD-160哈希对象

### update

更新哈希计算状态

#### 函数原型

```lua
function ripemd160:update(data)
end
```

#### 参数说明

* data
  要添加到哈希计算中的数据字符串

### digest

获取二进制哈希摘要

#### 函数原型

```lua
function ripemd160:digest()
end
```

#### 返回值

返回20字节的二进制字符串

### hexdigest

获取十六进制哈希摘要

#### 函数原型

```lua
function ripemd160:hexdigest()
end
```

#### 返回值

返回40个字符的十六进制字符串（大写）

### copy

复制哈希对象及其状态

#### 函数原型

```lua
function ripemd160:copy()
end
```

#### 返回值

返回包含相同内部状态的新RIPEMD-160对象

## 使用示例

### 基础哈希计算

```lua
local ripemd160 = require 'hashings.ripemd160'

-- 计算字符串的RIPEMD-160哈希
local function hash_string(str)
    local h = ripemd160:new(str)
    return h:hexdigest()
end

local hash = hash_string('Hello World')
print('RIPEMD-160:', hash)
```

### 比特币地址生成

```lua
local ripemd160 = require 'hashings.ripemd160'
local sha256 = require 'hashings.sha256'

-- 简化的比特币地址生成示例
local function bitcoin_address(public_key)
    -- 1. SHA-256哈希公钥
    local sha_hash = sha256:new(public_key):digest()

    -- 2. RIPEMD-160哈希上一步结果
    local ripemd_hash = ripemd160:new(sha_hash):digest()

    -- 3. 添加版本字节（主网为0x00）
    local versioned = string.char(0x00) .. ripemd_hash

    -- 4. 双重SHA-256计算校验和
    local checksum = sha256:new(sha256:new(versioned):digest()):digest():sub(1, 4)

    -- 5. 组合并Base58编码（需要Base58库）
    local address_bytes = versioned .. checksum
    -- return base58_encode(address_bytes)

    return address_bytes  -- 返回字节表示
end

-- 使用（需要实际的公钥）
local pubkey = 'PUBLIC_KEY_HERE'
local addr = bitcoin_address(pubkey)
print('Address bytes:', #addr)
```

### 数据完整性验证

```lua
local ripemd160 = require 'hashings.ripemd160'

local function verify_data(data, expected_hash)
    local h = ripemd160:new(data)
    local calculated_hash = h:hexdigest()

    return calculated_hash == expected_hash
end

local data = 'important data'
local hash = ripemd160:new(data):hexdigest()

if verify_data(data, hash) then
    print('Data integrity verified')
end
```

### 双重哈希

```lua
local ripemd160 = require 'hashings.ripemd160'
local sha256 = require 'hashings.sha256'

-- 哈希160 = RIPEMD-160(SHA-256(data))
local function hash160(data)
    local sha = sha256:new(data):digest()
    return ripemd160:new(sha):hexdigest()
end

-- 使用在比特币等系统中
local pubkey = 'PUBLIC_KEY_DATA'
local addr_hash = hash160(pubkey)
print('Hash160:', addr_hash)
```

## 算法原理

RIPEMD-160使用两个并行的处理路径：

1. **并行处理**：数据通过两个不同的压缩函数
2. **5轮操作**：每轮使用不同的非线性函数
3. **160位状态**：5个32位变量（A, B, C, D, E）
4. **最终组合**：两个路径的结果合并

### 非线性函数

```
f(j, x, y, z) = x ⊕ y ⊕ z                    (0 ≤ j < 16)
f(j, x, y, z) = (x ∧ y) ∨ (¬x ∧ z)            (16 ≤ j < 32)
f(j, x, y, z) = (x ∨ ¬y) ⊕ z                  (32 ≤ j < 48)
f(j, x, y, z) = (x ∧ z) ∨ (y ∧ ¬z)            (48 ≤ j < 64)
f(j, x, y, z) = x ⊕ (y ∨ ¬z)                  (64 ≤ j < 80)
```

## 性能特点

- **计算速度**：中等（比SHA-256慢）
- **内存占用**：低（20字节状态 + 64字节缓冲区）
- **适用场景**：密码学应用、比特币

## 类属性

```lua
ripemd160.digest_size  -- 20 (字节)
ripemd160.block_size   -- 64 (字节)
```

## 安全性评估

### 优势

- 比MD5和SHA-1更安全
- 没有已实用的碰撞攻击
- 在比特币中经过长期验证

### 局限性

- 160位摘要相对较短
- 不如SHA-256安全
- 速度较慢

## 使用场景

### 适合使用

✅ **推荐用于**：
- 比特币及相关应用
- 需要与比特币兼容的系统
- 数字签名系统
- 现有RIPEMD-160基础设施

### 不推荐使用

❌ **不推荐用于**：
- 新项目（用SHA-256）
- 需要最高安全性的场景
- 性能敏感应用

## 完整示例

### 简单的数字签名准备

```lua
local ripemd160 = require 'hashings.ripemd160'

-- 准备数字签名的消息哈希
local function prepare_message(message)
    local h = ripemd160:new()
    h:update(message)
    return h:digest()
end

-- 使用
local message = 'Important document to sign'
local msg_hash = prepare_message(message)

print('Message hash length:', #msg_hash)
print('Message hash (hex):', ripemd160:new(message):hexdigest())
```

### 文件校验

```lua
local ripemd160 = require 'hashings.ripemd160'

local function file_hash(filename)
    local file = io.open(filename, 'rb')
    if not file then
        return nil, 'Cannot open file'
    end

    local h = ripemd160:new()
    local total = 0

    while true do
        local block = file:read(64 * 1024)
        if not block then break end
        h:update(block)
        total = total + #block
    end

    file:close()

    return h:hexdigest(), total
end

local hash, size = file_hash('document.txt')
if hash then
    print(string.format('File: %d bytes, RIPEMD-160: %s', size, hash))
end
```

### 密钥派生

```lua
local ripemd160 = require 'hashings.ripemd160'

-- 从主密钥派生多个子密钥
local function derive_keys(master_key, contexts)
    local derived = {}

    for _, context in ipairs(contexts) do
        local h = ripemd160:new()
        h:update(master_key)
        h:update(context)
        derived[context] = h:hexdigest()
    end

    return derived
end

-- 使用
local master = 'MASTER_SECRET_KEY'
local keys = derive_keys(master, {
    'encryption',
    'signing',
    'authentication'
})

for context, key in pairs(keys) do
    print(context .. ':', key)
end
```

## 与SHA-256对比

### 迁移到SHA-256

```lua
-- 之前（RIPEMD-160）
local ripemd160 = require 'hashings.ripemd160'
local hash = ripemd160:new(data):hexdigest()  -- 40字符

-- 之后（SHA-256）
local sha256 = require 'hashings.sha256'
local hash = sha256:new(data):hexdigest()    -- 64字符
-- 更安全，但输出长度不同
```

## 注意事项

⚠️ **重要**：

1. **安全边际**：160位相对较短
2. **新项目**：新项目应考虑SHA-256
3. **兼容性**：主要用于比特币兼容性
4. **性能**：比SHA-256慢
5. **标准检查**：确认应用需要RIPEMD-160

## 最佳实践

1. **特定场景**：仅在需要比特币兼容性时使用
2. **考虑升级**：新项目考虑SHA-256
3. **组合使用**：通常与SHA-256组合使用
4. **安全评估**：评估160位是否足够
5. **性能测试**：在实际环境中测试性能
