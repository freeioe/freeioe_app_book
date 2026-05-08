---

# Keccak哈希算法 (hashings.keccak)

Keccak是基于海绵构造（sponge construction）的哈希函数家族，是SHA-3标准的基础。支持可变长度输出，提供高度灵活性和安全性。

## 特性

- **摘要长度**：可变（取决于参数）
- **块大小**：可变（取决于参数）
- **输出格式**：可配置的十六进制字符串
- **安全性**：极高，抗侧信道攻击
- **结构**：海绵构造（sponge construction）

## 与SHA-3的关系

Keccak是SHA-3标准的基础算法。SHA-3对Keccak进行了部分参数调整：

| 特性 | Keccak | SHA-3 |
| :--- | :--- | :--- |
| 算法 | Keccak | Keccak |
| 填充 | pad10*1 | pad10*1 |
| 输出长度 | 任意 | 固定 |
| 标准化 | 原始 | NIST标准 |

## 使用方法

```lua
local keccak = require 'hashings.keccak'

-- 创建Keccak对象（指定块大小和摘要大小）
-- block_size: 块大小（字节），digest_size: 摘要大小（字节）
local h = keccak:new(136, 32, 'Hello World')  -- SHA3-256等效
print(h:hexdigest())

-- 流式处理
local h = keccak:new(136, 32)
h:update('First part ')
h:update('Second part')
print(h:hexdigest())
```

## 接口说明

### new

创建Keccak哈希对象

#### 函数原型

```lua
function keccak:new(block_size, digest_size, data)
end
```

#### 参数说明

* block_size
  块大小（字节），影响海绵构造的rate参数
  - SHA3-224: 144字节
  - SHA3-256: 136字节
  - SHA3-384: 104字节
  - SHA3-512: 72字节

* digest_size
  摘要大小（字节），决定输出长度
  - SHA3-224: 28字节
  - SHA3-256: 32字节
  - SHA3-384: 48字节
  - SHA3-512: 64字节

* data
  可选，初始数据字符串

#### 返回值

返回Keccak哈希对象

### update

更新哈希计算状态

#### 函数原型

```lua
function keccak:update(data)
end
```

#### 参数说明

* data
  要添加到哈希计算中的数据字符串

### digest

获取二进制哈希摘要

#### 函数原型

```lua
function keccak:digest()
end
```

#### 返回值

返回digest_size字节的二进制字符串

### hexdigest

获取十六进制哈希摘要

#### 函数原型

```lua
function keccak:hexdigest()
end
```

#### 返回值

返回digest_size*2个字符的十六进制字符串（大写）

### copy

复制Keccak对象及其状态

#### 函数原型

```lua
function keccak:copy()
end
```

#### 返回值

返回包含相同内部状态的新Keccak对象

## 使用示例

### SHA3-256等效

```lua
local keccak = require 'hashings.keccak'

-- SHA3-256: block_size=136, digest_size=32
local h = keccak:new(136, 32, 'Hello World')
print(h:hexdigest())
```

### SHA3-512等效

```lua
local keccak = require 'hashings.keccak'

-- SHA3-512: block_size=72, digest_size=64
local h = keccak:new(72, 64, 'Hello World')
print(h:hexdigest())
```

### 自定义输出长度

```lua
local keccak = require 'hashings.keccak'

-- 自定义摘要长度
local function custom_hash(data, output_len)
    local h = keccak:new(136, output_len, data)
    return h:hexdigest()
end

-- 生成16字节哈希
local hash = custom_hash('data', 16)
print('Custom hash:', hash)
```

### 可变长度输出

```lua
local keccak = require 'hashings.keccak'

-- 从相同数据生成不同长度的哈希
local data = 'Important data'

-- 生成多个哈希
local h = keccak:new(136, 32, data)
local hash256 = h:hexdigest()

local h = keccak:new(136, 64, data)
local hash512 = h:hexdigest()

print('Hash256:', hash256)
print('Hash512:', hash512)
```

## 算法原理

Keccak使用海绵构造：

1. **状态初始化**：1600位状态初始化为0
2. **吸收阶段**：输入数据块与状态异或，然后进行排列
3. **挤压阶段**：从状态读取输出，继续排列直到获得足够输出

### Keccak-f[1600]排列

```
θ (Theta) - 纵向扩散
ρ (Rho) - 位移
π (Pi) - 置换
χ (Chi) - 非线性混合
ι (Iota) - 轮常数加
```

## 参数配置

### SHA-3标准参数

| 算法 | 块大小 | 摘要大小 | 容量 |
| :--- | :--- | :--- | :--- |
| SHA3-224 | 144 | 28 | 448 |
| SHA3-256 | 136 | 32 | 512 |
| SHA3-384 | 104 | 48 | 768 |
| SHA3-512 | 72 | 64 | 1024 |

### Shake算法

| 算法 | 块大小 | 最小输出 | 容量 |
| :--- | :--- | :--- | :--- |
| SHAKE128 | 168 | 任意 | 256 |
| SHAKE256 | 136 | 任意 | 512 |

## 性能特点

- **计算速度**：快，硬件友好
- **内存占用**：中等（200字节状态）
- **并行性**：易于并行化
- **适用场景**：通用哈希、可变输出

## 类属性

Keccak类的属性取决于实例化参数：

```lua
-- 具体属性取决于new()时设置的参数
-- digest_size和block_size由构造函数参数决定
```

## 使用场景

### 适合使用

✅ **推荐用于**：
- 需要可变输出长度
- 新项目
- 高安全性要求
- 抗侧信道攻击需求
- 硬件实现

### 不适合使用

❌ **不推荐用于**：
- 需要与SHA-2保持兼容
- 受限于标准要求
- 极度性能敏感（考虑BLAKE2）

## 完整示例

### 密钥派生

```lua
local keccak = require 'hashings.keccak'

-- 使用Keccak进行密钥派生
local function derive_key(master_key, context, output_len)
    local h = keccak:new(136, output_len)
    h:update(master_key)
    h:update(context)
    return h:digest()
end

-- 派生不同长度的密钥
local master = 'MASTER_SECRET'

local enc_key = derive_key(master, 'encryption', 32)
local mac_key = derive_key(master, 'mac', 64)

print('Encryption key:', #enc_key, 'bytes')
print('MAC key:', #mac_key, 'bytes')
```

### 随机数生成

```lua
local keccak = require 'hashings.keccak'

-- 使用Keccak作为随机数生成器
local function keccak_rng(seed, length)
    local h = keccak:new(136, length)
    h:update(seed)
    h:update(os.clock())  -- 添加时间
    return h:digest()
end

-- 生成随机字节
local random_bytes = keccak_rng('seed', 32)
print('Random:', #random_bytes, 'bytes')
```

### 哈希链

```lua
local keccak = require 'hashings.keccak'

-- 构建Keccak哈希链
local function hash_chain(initial, count)
    local current = initial

    for i = 1, count do
        local h = keccak:new(136, 32, current)
        current = h:digest()
    end

    return current
end

-- 使用
local seed = 'SEED_VALUE'
local final = hash_chain(seed, 1000)
print('Chain result:', #final, 'bytes')
```

### 流式哈希

```lua
local keccak = require 'hashings.keccak'

-- 处理大文件
local function file_keccak(filename, block_size, digest_size)
    local file = io.open(filename, 'rb')
    if not file then
        return nil, 'Cannot open file'
    end

    local h = keccak:new(block_size, digest_size)
    local total = 0

    while true do
        local block = file:read(block_size)
        if not block then break end
        h:update(block)
        total = total + #block
    end

    file:close()

    return h:hexdigest(), total
end

-- 使用
local hash, size = file_keccak('largefile.bin', 136, 32)
if hash then
    print(string.format('File: %d bytes, Keccak: %s', size, hash))
end
```

## 与SHA-3对比

### 使用原始Keccak

```lua
local keccak = require 'hashings.keccak'

-- 原始Keccak参数
local h = keccak:new(136, 32, 'data')
```

### 使用SHA-3包装器

```lua
local sha3_256 = require 'hashings.sha3_256'

-- SHA-3标准包装
local h = sha3_256:new('data')
```

## 注意事项

⚠️ **重要**：

1. **参数选择**：正确选择block_size和digest_size
2. **标准差异**：与SHA-3存在细微差异
3. **填充规则**：使用正确的填充规则
4. **输出长度**：digest_size决定输出长度
5. **性能考虑**：参数影响性能

## 最佳实践

1. **参数匹配**：使用标准参数组合
2. **标准优先**：优先使用SHA-3包装器
3. **可变输出**：充分利用可变输出特性
4. **安全评估**：评估参数选择的安全性
5. **测试验证**：在实际环境中测试

## 迁移建议

### 从Keccak到SHA-3

```lua
-- 原始Keccak
local keccak = require 'hashings.keccak'
local h = keccak:new(136, 32, data)

-- SHA-3包装器（推荐）
local sha3_256 = require 'hashings.sha3_256'
local h = sha3_256:new(data)
```

### 参数配置参考

```lua
-- SHA3-256等效
keccak:new(136, 32, data)

-- SHA3-512等效
keccak:new(72, 64, data)

-- SHAKE128等效（可变输出）
keccak:new(168, desired_length, data)
```
