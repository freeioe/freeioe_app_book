---

# Whirlpool哈希算法 (hashings.whirlpool)

Whirlpool是一种产生512位哈希值的加密哈希函数，基于宽踪策略设计。它使用专用的分组密码结构，提供高安全性和抗碰撞性。

## 特性

- **摘要长度**：512位（64字节）
- **块大小**：64字节（512位）
- **输出格式**：通常表示为128个十六进制字符
- **安全性**：极高，抗所有已知攻击
- **结构**：基于Merkle-Damgård结构的专用分组密码

## 与其他哈希算法比较

| 算法 | 摘要长度 | 速度 | 安全性 | 应用场景 |
| :--- | :--- | :--- | :--- | :--- |
| SHA-256 | 256位 | 中等 | 高 | 通用 |
| SHA-512 | 512位 | 快 | 极高 | 大文件 |
| SHA3-512 | 512位 | 快 | 极高 | 新标准 |
| **Whirlpool** | **512位** | **中等** | **极高** | **高安全需求** |

## 使用方法

```lua
local whirlpool = require 'hashings.whirlpool'

-- 创建Whirlpool哈希对象并计算哈希
local h = whirlpool:new('Hello World')
print(h:hexdigest())  -- 输出: 128位十六进制字符串

-- 流式处理
local h = whirlpool:new()
h:update('First part ')
h:update('Second part')
print(h:hexdigest())
```

## 接口说明

### new

创建Whirlpool哈希对象

#### 函数原型

```lua
function whirlpool:new(data)
end
```

#### 参数说明

* data
  可选，初始数据字符串

#### 返回值

返回Whirlpool哈希对象

### update

更新哈希计算状态

#### 函数原型

```lua
function whirlpool:update(data)
end
```

#### 参数说明

* data
  要添加到哈希计算中的数据字符串

### digest

获取二进制哈希摘要

#### 函数原型

```lua
function whirlpool:digest()
end
```

#### 返回值

返回64字节的二进制字符串

### hexdigest

获取十六进制哈希摘要

#### 函数原型

```lua
function whirlpool:hexdigest()
end
```

#### 返回值

返回128个字符的十六进制字符串（大写）

### copy

复制哈希对象及其状态

#### 函数原型

```lua
function whirlpool:copy()
end
```

#### 返回值

返回包含相同内部状态的新Whirlpool对象

## 使用示例

### 高安全性数据校验

```lua
local whirlpool = require 'hashings.whirlpool'

-- 用于需要最高安全性的场景
local function high_security_hash(data)
    local h = whirlpool:new()
    h:update(data)
    return h:hexdigest()
end

local critical_data = 'CRITICAL_SYSTEM_DATA'
local hash = high_security_hash(critical_data)
print('Whirlpool hash:', hash)
```

### 文件指纹

```lua
local whirlpool = require 'hashings.whirlpool'

local function file_fingerprint(filename)
    local file = io.open(filename, 'rb')
    if not file then
        return nil, 'Cannot open file'
    end

    local h = whirlpool:new()
    local file_size = 0

    while true do
        local block = file:read(64 * 1024)
        if not block then break end
        h:update(block)
        file_size = file_size + #block
    end

    file:close()

    return h:hexdigest(), file_size
end

local fp, size = file_fingerprint('important_file.bin')
if fp then
    print(string.format('File: %d bytes', size))
    print('Whirlpool:', fp)
end
```

### 密钥派生

```lua
local whirlpool = require 'hashings.whirlpool'

-- 从主密钥派生强密钥
local function derive_strong_key(master_key, context, iterations)
    local h = whirlpool:new()

    local data = master_key .. context
    for i = 1, iterations do
        h = whirlpool:new(h:digest() .. data)
    end

    return h:digest()
end

-- 派生加密密钥
local master = 'MASTER_KEY_HERE'
local enc_key = derive_strong_key(master, 'encryption', 100000)
print('Derived key length:', #enc_key)
```

### 数字签名准备

```lua
local whirlpool = require 'hashings.whirlpool'

-- 准备用于数字签名的消息摘要
local function prepare_signature(message)
    local h = whirlpool:new()
    h:update(message)
    return h:digest()
end

-- 使用
local document = 'Important legal document'
local digest = prepare_signature(document)

print('Digest length:', #digest)
print('Digest (hex):', whirlpool:new(document):hexdigest())
```

## 算法原理

Whirlpool使用专用的分组密码作为压缩函数：

1. **分组密码**：基于替代表和置换的分组密码
2. **Miyaguchi-Preneel**：使用Miyaguchi-Preneel压缩模式
3. **512位状态**：维护512位内部状态
4. **S盒替换**：使用8个不同的256字节S盒

### 算法结构

```
Whirlpool压缩函数：
- 输入：512位状态 + 512位消息块
- 处理：10轮分组密码操作
- 输出：新的512位状态

最终输出：512位状态
```

## 性能特点

- **计算速度**：中等（比SHA-512慢）
- **内存占用**：中等（64字节状态 + S盒）
- **适用场景**：高安全性要求

## 类属性

```lua
whirlpool.digest_size  -- 64 (字节)
whirlpool.block_size   -- 64 (字节)
```

## 安全性评估

### 优势

- **512位输出**：极大的搜索空间
- **抗碰撞**：无已知有效碰撞攻击
- **抗预像**：无已知预像攻击
- **设计简单**：算法结构清晰

### 性能考虑

- 速度比SHA-512慢
- S盒占用内存
- 适合安全性优先的场景

## 使用场景

### 适合使用

✅ **推荐用于**：
- 最高安全性要求
- 长期数据保护
- 数字签名系统
- 密码学原语
- 抗量子计算考虑

### 不适合使用

❌ **不推荐用于**：
- 性能敏感应用（用SHA-512）
- 资源受限系统
- 需要广泛支持的场景

## 完整示例

### 安全存储验证

```lua
local whirlpool = require 'hashings.whirlpool'

-- 安全文件存储系统
local function store_secure(data)
    -- 计算哈希
    local h = whirlpool:new(data)
    local hash = h:hexdigest()

    -- 存储数据和哈希
    return {
        data = data,
        hash = hash,
        timestamp = os.time()
    }
end

local function verify_secure(record)
    local h = whirlpool:new(record.data)
    local calculated = h:hexdigest()

    if calculated ~= record.hash then
        return false, 'Data corrupted'
    end

    return true, record.data
end

-- 使用
local record = store_secure('CRITICAL_DATA')
local valid, data_or_err = verify_secure(record)

if valid then
    print('Data verified:', data_or_err)
else
    print('Verification failed:', data_or_err)
end
```

### 多层哈希

```lua
local whirlpool = require 'hashings.whirlpool'
local sha256 = require 'hashings.sha256'

-- 多层哈希系统
local function multi_layer_hash(data)
    return {
        sha256 = sha256:new(data):hexdigest(),      -- 快速检查
        whirlpool = whirlpool:new(data):hexdigest() -- 高安全验证
    }
end

local function verify_multi_layer(data, hashes)
    -- 快速检查
    local calc_sha256 = sha256:new(data):hexdigest()
    if calc_sha256 ~= hashes.sha256 then
        return false, 'SHA-256 mismatch'
    end

    -- 高安全验证
    local calc_whirlpool = whirlpool:new(data):hexdigest()
    if calc_whirlpool ~= hashes.whirlpool then
        return false, 'Whirlpool mismatch'
    end

    return true
end

-- 使用
local data = 'IMPORTANT_DATA'
local hashes = multi_layer_hash(data)

print('SHA-256:', hashes.sha256)
print('Whirlpool:', hashes.whirlpool)

local valid, err = verify_multi_layer(data, hashes)
if valid then
    print('All verifications passed')
end
```

### 数据去重

```lua
local whirlpool = require 'hashings.whirlpool'

-- 使用Whirlpool进行数据去重
local function deduplicate(dataset)
    local seen = {}
    local unique = {}

    for _, item in ipairs(dataset) do
        local hash = whirlpool:new(item):hexdigest()

        if not seen[hash] then
            seen[hash] = true
            table.insert(unique, item)
        end
    end

    return unique
end

-- 使用
local data = {
    'item1',
    'item2',
    'item1',  -- 重复
    'item3',
    'item2'   -- 重复
}

local unique_data = deduplicate(data)
print('Original:', #data, 'items')
print('Unique:', #unique_data, 'items')
```

## 与SHA-512比较

### 选择Whirlpool的场景

- **需要最高安全性**
- **512位输出要求**
- **特定应用需求**

### 选择SHA-512的场景

- **性能优先**
- **广泛支持**
- **标准兼容**

## 注意事项

⚠️ **重要**：

1. **性能考虑**：比SHA-512慢
2. **支持有限**：不如SHA家族广泛支持
3. **内存占用**：S盒需要额外内存
4. **标准检查**：确认应用环境支持Whirlpool
5. **安全边际**：512位提供极大安全边际

## 最佳实践

1. **高安全场景**：用于需要最高安全性的应用
2. **性能权衡**：评估性能需求
3. **标准优先**：优先考虑SHA-3或SHA-512
4. **测试验证**：在实际环境中测试
5. **文档记录**：明确使用Whirlpool的原因

## 迁移建议

### 从Whirlpool迁移到SHA-512

```lua
-- 之前（Whirlpool）
local whirlpool = require 'hashings.whirlpool'
local hash = whirlpool:new(data):hexdigest()  -- 128字符

-- 之后（SHA-512）
local sha512 = require 'hashings.sha512'
local hash = sha512:new(data):hexdigest()     -- 128字符
-- 相同输出长度，但更快且更广泛支持
```

### 迁移到SHA3-512

```lua
-- 之前（Whirlpool）
local whirlpool = require 'hashings.whirlpool'

-- 之后（SHA3-512）
local sha3_512 = require 'hashings.sha3_512'
-- 相同输出长度，SHA-3标准，更安全
```
