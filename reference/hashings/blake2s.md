---

# BLAKE2s哈希算法 (hashings.blake2s)

BLAKE2s是BLAKE2加密哈希函数的32位优化版本，产生可变长度的哈希值（最大32字节）。速度快且安全性高。

## 特性

- **摘要长度**：可变，最大32字节（默认256位）
- **块大小**：64字节（512位）
- **输出格式**：通常表示为最多64个十六进制字符
- **安全性**：高，抗侧信道攻击
- **性能**：极快，专为32位系统优化

## 与BLAKE2b的区别

| 特性 | BLAKE2s | BLAKE2b |
| :--- | :--- | :--- |
| 字长 | 32位 | 64位 |
| 最大摘要 | 32字节 | 64字节 |
| 块大小 | 64字节 | 128字节 |
| 目标平台 | 32位CPU | 64位CPU |
| 轮数 | 10轮 | 12/14/16轮 |

## 使用方法

```lua
local blake2s = require 'hashings.blake2s'

-- 创建BLAKE2s哈希对象并计算哈希
local h = blake2s:new('Hello World')
print(h:hexdigest())  -- 输出: 64位十六进制字符串

-- 流式处理
local h = blake2s:new()
h:update('First part ')
h:update('Second part')
print(h:hexdigest())
```

## 接口说明

### new

创建BLAKE2s哈希对象

#### 函数原型

```lua
function blake2s:new(data)
end
```

#### 参数说明

* data
  可选，初始数据字符串

#### 返回值

返回BLAKE2s哈希对象

### update

更新哈希计算状态

#### 函数原型

```lua
function blake2s:update(data)
end
```

#### 参数说明

* data
  要添加到哈希计算中的数据字符串

### digest

获取二进制哈希摘要

#### 函数原型

```lua
function blake2s:digest()
end
```

#### 返回值

返回32字节的二进制字符串

### hexdigest

获取十六进制哈希摘要

#### 函数原型

```lua
function blake2s:hexdigest()
end
```

#### 返回值

返回64个字符的十六进制字符串（大写）

### copy

复制哈希对象及其状态

#### 函数原型

```lua
function blake2s:copy()
end
```

#### 返回值

返回包含相同内部状态的新BLAKE2s对象

## 使用示例

### 高性能哈希

```lua
local blake2s = require 'hashings.blake2s'

-- BLAKE2s非常快速，适合性能敏感场景
local function fast_hash(data)
    local h = blake2s:new(data)
    return h:hexdigest()
end

-- 批量处理
for i = 1, 10000 do
    local hash = fast_hash('data_' .. i)
    -- 处理哈希...
end
```

### 数据校验

```lua
local blake2s = require 'hashings.blake2s'

local function checksum_file(filename)
    local file = io.open(filename, 'rb')
    if not file then return nil end

    local h = blake2s:new()
    while true do
        local block = file:read(64 * 1024)
        if not block then break end
        h:update(block)
    end
    file:close()

    return h:hexdigest()
end
```

### 密钥派生

```lua
local blake2s = require 'hashings.blake2s'

-- BLAKE2s速度快，适合密钥派生
local function derive_key(master_key, context, iterations)
    local h = blake2s:new()

    local data = master_key .. context
    for i = 1, iterations do
        h = blake2s:new(h:digest() .. data)
    end

    return h:digest()
end

local derived_key = derive_key('master_key', 'encryption', 10000)
```

## 性能特点

- **计算速度**：极快，比SHA-256快得多
- **内存占用**：低（64字节缓冲区 + 8个32位状态变量）
- **适用场景**：32位系统、性能敏感应用

## 类属性

```lua
blake2s.digest_size  -- 32 (字节)
blake2s.block_size   -- 64 (字节)
```

## BLAKE2优势

### 相比SHA-256

1. **更快**：BLAKE2s比SHA-256快得多
2. **更安全**：抗侧信道攻击
3. **更灵活**：支持可变摘要长度
4. **更简单**：算法结构更简单

### 相比SHA-3

1. **更快**：BLAKE2s比SHA3-256快
2. **成熟度**：在实际应用中验证更多
3. **广泛支持**：许多现代库都支持

## 平台选择

### 32位系统

✅ **推荐使用BLAKE2s**：
- ARM Cortex-M
- 嵌入式32位处理器
- 性能受限的设备

### 64位系统

考虑使用**BLAKE2b**：
- x86-64处理器
- ARM64处理器
- 可以处理64位操作

## 安全应用

### 密码存储

```lua
local blake2s = require 'hashings.blake2s'

-- BLAKE2s速度快，但仍需配合盐和迭代
local function secure_hash(password, salt)
    local h = blake2s:new()
    h:update(salt)
    h:update(password)

    -- 多轮迭代
    local result = h:digest()
    for i = 2, 10000 do
        h = blake2s:new(result .. salt)
        result = h:digest()
    end

    return h:hexdigest()
end
```

### 消息认证

```lua
local blake2s = require 'hashings.blake2s'

local function mac(key, message)
    local h = blake2s:new()
    h:update(key)
    h:update(message)
    return h:hexdigest()
end
```

## 性能基准

### 与其他算法比较

在32位系统上的相对性能（以SHA-256为基准）：

| 算法 | 相对速度 |
| :--- | :--- |
| BLAKE2s | 3-5倍快 |
| SHA-256 | 基准 |
| SHA3-256 | 比SHA-256慢 |
| SHA-512 | 很慢（32位） |

### 实际测试

```lua
local blake2s = require 'hashings.blake2s'
local sha256 = require 'hashings.sha256'

local function benchmark(hash_func, data, iterations)
    local start = os.clock()

    for i = 1, iterations do
        hash_func:new(data):hexdigest()
    end

    return os.clock() - start
end

local data = string.rep('test data ', 100)
local blake2s_time = benchmark(blake2s, data, 1000)
local sha256_time = benchmark(sha256, data, 1000)

print(string.format('BLAKE2s: %.4f seconds', blake2s_time))
print(string.format('SHA-256: %.4f seconds', sha256_time))
print(string.format('Speedup: %.2fx', sha256_time / blake2s_time))
```

## 使用建议

### 选择BLAKE2s的场景

✅ **推荐**：
- 32位嵌入式系统
- 性能敏感应用
- 需要快速哈希的场景
- 新项目（没有历史兼容要求）

### 选择BLAKE2b的场景

✅ **推荐**：
- 64位系统
- 需要更长摘要（>32字节）
- 批量数据处理

### 不推荐BLAKE2s

❌ **不推荐**：
- 需要与SHA-256保持一致的输出格式
- 64位高性能系统（用BLAKE2b）
- 受限于标准要求的协议

## 完整示例

### 嵌入式系统校验

```lua
local blake2s = require 'hashings.blake2s'

-- 固件校验
local function verify_firmware(firmware_data)
    local h = blake2s:new()
    h:update(firmware_data)
    local hash = h:hexdigest()

    -- 比较预存的哈希
    local expected_hash = 'PRECOMPUTED_HASH'
    return hash == expected_hash
end
```

### 缓存键生成

```lua
local blake2s = require 'hashings.blake2s'

-- BLAKE2s快速，适合缓存键生成
local function cache_key(key_part1, key_part2)
    local h = blake2s:new()
    h:update(key_part1)
    h:update('|')
    h:update(key_part2)
    return h:hexdigest():sub(1, 32)  -- 使用前32字符
end
```

## 注意事项

⚠️ **重要**：

1. **实现特定**：这是BLAKE2s，不是BLAKE2b
2. **字长限制**：在64位系统上考虑使用BLAKE2b
3. **摘要长度**：默认固定32字节（256位）
4. **标准检查**：确认应用环境支持BLAKE2

## 最佳实践

1. **性能优先**：在性能敏感场景优先考虑
2. **平台匹配**：32位系统用BLAKE2s，64位用BLAKE2b
3. **新项目**：新项目优先考虑BLAKE2而非SHA-2
4. **安全测试**：在部署前进行安全评估
