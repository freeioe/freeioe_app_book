---

# CRC32校验和 (hashings.crc32)

CRC32（Cyclic Redundancy Check 32-bit）是一种32位的循环冗余校验算法，广泛用于数据完整性校验和错误检测。

## 特性

- **输出长度**：32位（4字节）
- **块大小**：8位（逐字节处理）
- **输出格式**：通常表示为8个十六进制字符
- **用途**：数据校验，非加密用途

⚠️ **重要提示**：CRC32不是加密哈希函数，**不能用于安全目的**。它仅适用于数据完整性校验和错误检测。

## 使用方法

```lua
local crc32 = require 'hashings.crc32'

-- 计算CRC32校验和
local h = crc32:new('Hello World')
print(h:hexdigest())  -- 输出: 8DD13588

-- 流式处理
local h = crc32:new()
h:update('First part ')
h:update('Second part')
print(h:hexdigest())

-- 获取二进制输出
local h = crc32:new('Hello World')
local binary = h:digest()  -- 4字节二进制数据
```

## 接口说明

### new

创建CRC32对象

#### 函数原型

```lua
function crc32:new(data)
end
```

#### 参数说明

* data
  可选，初始数据字符串

#### 返回值

返回CRC32对象

### update

更新CRC32计算

#### 函数原型

```lua
function crc32:update(data)
end
```

#### 参数说明

* data
  要添加到计算中的数据字符串

### digest

获取二进制校验和

#### 函数原型

```lua
function crc32:digest()
end
```

#### 返回值

返回4字节的二进制字符串

### hexdigest

获取十六进制校验和

#### 函数原型

```lua
function crc32:hexdigest()
end
```

#### 返回值

返回8个字符的十六进制字符串（大写）

### copy

复制CRC32对象及其状态

#### 函数原型

```lua
function crc32:copy()
end
```

#### 返回值

返回包含相同内部状态的新CRC32对象

## 使用示例

### 文件校验和

```lua
local crc32 = require 'hashings.crc32'

local function file_crc32(filename)
    local file = io.open(filename, 'rb')
    if not file then return nil, 'Cannot open file' end

    local h = crc32:new()
    while true do
        local block = file:read(64 * 1024)  -- 64KB块
        if not block then break end
        h:update(block)
    end
    file:close()
    return h:hexdigest()
end

print('File CRC32:', file_crc32('data.bin'))
```

### 数据传输校验

```lua
local crc32 = require 'hashings.crc32'

local function send_with_checksum(socket, data)
    -- 计算校验和
    local h = crc32:new(data)
    local checksum = h:digest()

    -- 发送数据和校验和
    socket:send(data)
    socket:send(checksum)
end

local function receive_with_checksum(socket)
    -- 接收数据
    local data = socket:receive()
    local checksum = socket:receive(4)  -- 4字节校验和

    -- 验证校验和
    local h = crc32:new(data)
    local calculated = h:digest()

    if calculated == checksum then
        return data, true
    else
        return nil, false  -- 校验失败
    end
end
```

### ZIP文件校验

CRC32广泛用于ZIP文件格式的校验：

```lua
local crc32 = require 'hashings.crc32'

local function calculate_entry_crc(file_data)
    local h = crc32:new(file_data)
    return h:hexdigest()  -- 用于ZIP本地文件头
end
```

### 增量校验

```lua
local crc32 = require 'hashings.crc32'

-- 创建校验点
local h1 = crc32:new()
h1:update('Part 1 ')

local h2 = h1:copy()  -- 保存状态
h1:update('Part 2')

-- 从保存点继续
h2:update('Part 2')

print(h1:hexdigest())  -- 相同输出
print(h2:hexdigest())  -- 相同输出
```

## 性能特点

- **计算速度**：非常快
- **内存占用**：极低（仅需4字节状态）
- **适用场景**：数据完整性校验，错误检测

## 类属性

```lua
crc32.digest_size  -- 4 (字节)
crc32.block_size   -- 8 (位，逐字节处理)
```

## 与其他校验和比较

| 算法 | 长度 | 速度 | 碰撞概率 | 用途 |
| :--- | :--- | :--- | :--- | :--- |
| **CRC32** | **32位** | **快** | **中等** | **通用校验** |
| CRC16 | 16位 | 最快 | 高 | 简单协议 |
| Adler32 | 32位 | 最快 | 高 | Zlib压缩 |
| MD5 | 128位 | 中等 | 极低 | 文件指纹 |
| SHA-256 | 256位 | 慢 | 可忽略 | 安全用途 |

## CRC32的特点

### 优点

✅ **计算速度快**：简单的位运算
✅ **内存占用小**：仅需4字节状态
✅ **硬件支持**：许多CPU有CRC32指令
✅ **广泛应用**：ZIP、以太网、存储设备等

### 缺点

❌ **不安全**：不是加密哈希，容易伪造
❌ **碰撞概率**：32位存在碰撞可能
❌ **错误检测**：不是100%可靠（极小概率会漏检）

## 常见应用场景

### 1. 文件传输校验

```lua
-- 发送端
local crc32 = require 'hashings.crc32'

local function send_file(file, socket)
    local h = crc32:new()
    for chunk in file:chunks(8192) do
        socket:send(chunk)
        h:update(chunk)
    end
    socket:send(h:digest())
end

-- 接收端
local function receive_file(socket, output_file)
    local h = crc32:new()
    while true do
        local chunk = socket:receive(8192)
        if chunk == '' then break end
        output_file:write(chunk)
        h:update(chunk)
    end

    local checksum = socket:receive(4)
    return h:digest() == checksum
end
```

### 2. 数据库校验

```lua
local crc32 = require 'hashings.crc32'

local function record_checksum(record)
    local data = table.concat(record, '|')
    local h = crc32:new(data)
    return h:hexdigest()
end

-- 存储时计算
local checksum = record_checksum({id=1, name='test', value=100})

-- 验证时重新计算
local verify_checksum = record_checksum({id=1, name='test', value=100})
```

### 3. 协议实现

```lua
local crc32 = require 'hashings.crc32'

local function build_packet(data)
    local h = crc32:new()
    h:update(data)

    return data .. h:digest()  -- 数据 + 校验和
end

local function verify_packet(packet)
    local data = packet:sub(1, -5)  -- 除去最后4字节
    local checksum = packet:sub(-4)  -- 最后4字节

    local h = crc32:new(data)
    return h:digest() == checksum
end
```

## 数学原理

CRC32基于多项式除法原理：

1. 将数据视为二进制多项式
2. 使用生成多项式（IEEE 802.3标准：0xEDB88320）进行模2除法
3. 余数即为CRC32值

生成多项式（反转）：`0xEDB88320`

## 注意事项

⚠️ **重要**：
1. **非加密用途**：不要用于密码存储或消息认证
2. **碰撞风险**：大量数据时存在碰撞可能
3. **错误检测**：理论上存在检测失败的概率（极低）
4. **标准选择**：不同应用使用不同标准，确保匹配
