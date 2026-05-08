---

# BLAKE2b哈希算法 (hashings.blake2b)

BLAKE2b是BLAKE2加密哈希函数的64位优化版本，产生可变长度的哈希值（最大64字节）。速度极快且安全性高，是BLAKE2s的64位版本。

## 特性

- **摘要长度**：可变，最大64字节（默认512位）
- **块大小**：128字节（1024位）
- **输出格式**：通常表示为最多128个十六进制字符
- **安全性**：高，抗侧信道攻击
- **性能**：极快，专为64位系统优化

## 与BLAKE2s的区别

| 特性 | BLAKE2b | BLAKE2s |
| :--- | :--- | :--- |
| 字长 | 64位 | 32位 |
| 最大摘要 | 64字节 | 32字节 |
| 块大小 | 128字节 | 64字节 |
| 目标平台 | 64位CPU | 32位CPU |
| 轮数 | 12轮 | 10轮 |

## 使用方法

```lua
local blake2b = require 'hashings.blake2b'

-- 创建BLAKE2b哈希对象并计算哈希
local h = blake2b:new('Hello World')
print(h:hexdigest())  -- 输出: 128位十六进制字符串

-- 流式处理
local h = blake2b:new()
h:update('First part ')
h:update('Second part')
print(h:hexdigest())
```

## 接口说明

### new

创建BLAKE2b哈希对象

#### 函数原型

```lua
function blake2b:new(data)
end
```

#### 参数说明

* data
  可选，初始数据字符串

#### 返回值

返回BLAKE2b哈希对象

### update

更新哈希计算状态

#### 函数原型

```lua
function blake2b:update(data)
end
```

#### 参数说明

* data
  要添加到哈希计算中的数据字符串

### digest

获取二进制哈希摘要

#### 函数原型

```lua
function blake2b:digest()
end
```

#### 返回值

返回64字节的二进制字符串

### hexdigest

获取十六进制哈希摘要

#### 函数原型

```lua
function blake2b:hexdigest()
end
```

#### 返回值

返回128个字符的十六进制字符串（大写）

### copy

复制哈希对象及其状态

#### 函数原型

```lua
function blake2b:copy()
end
```

#### 返回值

返回包含相同内部状态的新BLAKE2b对象

## 使用示例

### 高性能哈希

```lua
local blake2b = require 'hashings.blake2b'

-- BLAKE2b在64位系统上非常快速
local function fast_hash(data)
    local h = blake2b:new(data)
    return h:hexdigest()
end

-- 批量处理
for i = 1, 10000 do
    local hash = fast_hash('data_' .. i)
    -- 处理哈希...
end
```

### 数据完整性验证

```lua
local blake2b = require 'hashings.blake2b'

local function verify_file(filename, expected_hash)
    local file = io.open(filename, 'rb')
    if not file then
        return false, 'Cannot open file'
    end

    local h = blake2b:new()
    while true do
        local block = file:read(128 * 1024)
        if not block then break end
        h:update(block)
    end
    file:close()

    local calculated_hash = h:hexdigest()
    return calculated_hash == expected_hash, calculated_hash
end

-- 使用
local valid, hash_or_err = verify_file('important.dat',
    'EXPECTED_HASH_HERE')

if valid then
    print('File integrity verified')
else
    print('Verification failed:', hash_or_err)
end
```

### 密钥派生

```lua
local blake2b = require 'hashings.blake2b'

-- BLAKE2b适合密钥派生
local function derive_key(master_key, context, iterations)
    local h = blake2b:new()

    local data = master_key .. context
    for i = 1, iterations do
        h = blake2b:new(h:digest() .. data)
    end

    return h:digest()
end

-- 派生多个密钥
local master_key = 'super_secret_master_key'
local enc_key = derive_key(master_key, 'encryption', 10000)
local mac_key = derive_key(master_key, 'mac', 10000)

print('Encryption key length:', #enc_key)
print('MAC key length:', #mac_key)
```

## 性能特点

- **计算速度**：极快，在64位系统上比SHA-512快得多
- **内存占用**：低（128字节缓冲区 + 8个64位状态变量）
- **适用场景**：64位系统、高性能应用

## 类属性

```lua
blake2b.digest_size  -- 64 (字节)
blake2b.block_size   -- 128 (字节)
```

## 使用建议

### 选择BLAKE2b的场景

✅ **推荐**：
- 64位系统
- 高性能要求
- 需要快速哈希的场景
- 新项目（没有历史兼容要求）
- 需要长哈希摘要

### 不推荐BLAKE2b

❌ **不推荐**：
- 需要与SHA-256保持一致的输出格式
- 受限于标准要求的协议
- 32位系统（用BLAKE2s）

## 注意事项

⚠️ **重要**：

1. **实现特定**：这是BLAKE2b，不是BLAKE2s
2. **平台优化**：在64位系统上性能最佳
3. **摘要长度**：默认固定64字节（512位）
4. **标准检查**：确认应用环境支持BLAKE2

## 最佳实践

1. **64位优先**：在64位系统上优先考虑BLAKE2b
2. **性能优先**：在性能敏感场景优先考虑
3. **新项目**：新项目优先考虑BLAKE2而非SHA-2
4. **安全测试**：在部署前进行安全评估
5. **平台匹配**：64位用BLAKE2b，32位用BLAKE2s
