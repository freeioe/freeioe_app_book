---

# Adler32校验和 (hashings.adler32)

Adler32是一种快速校验和算法，由Mark Adler发明。它是Zlib压缩库的默认校验和算法。

## 特性

- **输出长度**：32位（4字节）
- **块大小**：8位（逐字节处理）
- **输出格式**：通常表示为8个十六进制字符
- **用途**：数据校验，非加密用途

## 与CRC32比较

| 特性 | Adler32 | CRC32 |
| :--- | :--- | :--- |
| 速度 | 极快 | 快 |
| 错误检测 | 较弱 | 较强 |
| 硬件支持 | 少 | 多 |
| 压缩库 | Zlib使用 | - |
| 适用场景 | 简单校验 | 可靠校验 |

## 使用方法

```lua
local adler32 = require 'hashings.adler32'

-- 计算Adler32校验和
local h = adler32:new('Hello World')
print(h:hexdigest())  -- 输出: 8位十六进制字符串

-- 流式处理
local h = adler32:new()
h:update('First part ')
h:update('Second part')
print(h:hexdigest())
```

## 接口说明

### new

创建Adler32对象

#### 函数原型

```lua
function adler32:new(data)
end
```

#### 参数说明

* data
  可选，初始数据字符串

#### 返回值

返回Adler32对象

### update

更新校验和计算

#### 函数原型

```lua
function adler32:update(data)
end
```

#### 参数说明

* data
  要添加到计算中的数据字符串

### digest

获取二进制校验和

#### 函数原型

```lua
function adler32:digest()
end
```

#### 返回值

返回4字节的二进制字符串

### hexdigest

获取十六进制校验和

#### 函数原型

```lua
function adler32:hexdigest()
end
```

#### 返回值

返回8个字符的十六进制字符串（大写）

### copy

复制Adler32对象及其状态

#### 函数原型

```lua
function adler32:copy()
end
```

#### 返回值

返回包含相同内部状态的新Adler32对象

## 使用示例

### 数据校验

```lua
local adler32 = require 'hashings.adler32'

local function verify_data(data, expected_checksum)
    local h = adler32:new(data)
    local checksum = h:hexdigest()
    return checksum == expected_checksum
end

local data = 'important data'
local checksum = adler32:new(data):hexdigest()

if verify_data(data, checksum) then
    print('Data verified successfully')
end
```

### 数据传输

```lua
local adler32 = require 'hashings.adler32'

-- 发送方：计算并附上校验和
local function send_with_checksum(socket, data)
    local h = adler32:new(data)
    local checksum = h:digest()

    socket:send(data)
    socket:send(checksum)  -- 4字节
end

-- 接收方：验证校验和
local function receive_with_checksum(socket)
    local data = socket:receive()
    local checksum = socket:receive(4)  -- 4字节

    local h = adler32:new(data)
    local calculated = h:digest()

    if calculated == checksum then
        return data, true
    else
        return nil, false
    end
end
```

### 与Zlib配合

```lua
local adler32 = require 'hashings.adler32'
local zlib = require 'zlib'

-- 压缩数据并计算Adler32
local function compress_with_checksum(data)
    local compressed = zlib.compress(data)
    local checksum = adler32:new(compressed):hexdigest()

    return compressed, checksum
end

-- 解压并验证
local function decompress_with_verify(compressed, expected_checksum)
    local checksum = adler32:new(compressed):hexdigest()

    if checksum ~= expected_checksum then
        return nil, 'Checksum mismatch'
    end

    return zlib.decompress(compressed)
end
```

## 算法原理

Adler32使用两个16位累加器：

```
A = 1 + D1 + D2 + ... + Dn  (mod 65521)
B = (1 + D1) + (2 + D2) + ... + (n + Dn)  (mod 65521)
Adler32 = B × 65536 + A
```

其中D1, D2, ..., Dn是数据字节。

### 性能特点

- **初始值**：A = 1, B = 0
- **模数**：65521（质数）
- **优势**：计算简单，无需查找表

## 性能比较

```lua
local adler32 = require 'hashings.adler32'
local crc32 = require 'hashings.crc32'

local function benchmark(hash_func, data, iterations)
    local start = os.clock()

    for i = 1, iterations do
        hash_func:new(data):hexdigest()
    end

    return os.clock() - start
end

local data = string.rep('test data ', 1000)
local adler_time = benchmark(adler32, data, 10000)
local crc_time = benchmark(crc32, data, 10000)

print(string.format('Adler32: %.4f seconds', adler_time))
print(string.format('CRC32:  %.4f seconds', crc_time))
print(string.format('Speedup: %.2fx', crc_time / adler_time))
```

Adler32通常比CRC32快2-3倍。

## 类属性

```lua
adler32.digest_size  -- 4 (字节)
adler32.block_size   -- 8 (位，逐字节处理)
```

## 使用场景

### 适合使用

✅ **推荐**：
- 压缩数据校验（Zlib标准）
- 简单数据验证
- 性能敏感场景
- 非关键数据的完整性检查

### 不适合使用

❌ **不推荐**：
- 需要强错误检测的场景
- 通信协议（用CRC32）
- 文件格式（用CRC32）
- 安全敏感应用

## 与其他算法对比

| 算法 | 长度 | 速度 | 错误检测 | 用途 |
| :--- | :--- | :--- | :--- | :--- |
| **Adler32** | **32位** | **最快** | **中等** | **Zlib** |
| CRC32 | 32位 | 快 | 好 | 通用校验 |
| CRC16 | 16位 | 极快 | 中等 | 简单协议 |
| CRC-CCITT | 16位 | 极快 | 中等 | X.25等 |

## 注意事项

⚠️ **重要**：

1. **错误检测**：弱于CRC32，不适合关键数据
2. **安全性**：不安全，不能用于加密目的
3. **碰撞概率**：随机数据碰撞概率为1/65521
4. **标准使用**：主要用于Zlib压缩库

## 完整示例

### 数据包校验

```lua
local adler32 = require 'hashings.adler32'

local function create_packet(data)
    local h = adler32:new(data)
    local checksum = h:digest()

    -- 数据包格式: [数据长度(4)][数据][校验和(4)]
    local packet = string.pack('<I', #data) .. data .. checksum
    return packet
end

local function parse_packet(packet)
    if #packet < 8 then
        return nil, 'Packet too short'
    end

    local data_len = string.unpack('<I', packet:sub(1, 4))
    local data = packet:sub(5, 4 + data_len)
    local checksum = packet:sub(5 + data_len)

    local h = adler32:new(data)
    local calculated = h:digest()

    if calculated ~= checksum then
        return nil, 'Checksum mismatch'
    end

    return data
end
```

### 流式数据处理

```lua
local adler32 = require 'hashings.adler32'

-- 增量更新校验和
local h = adler32:new()

h:update('chunk1')
h:update('chunk2')
h:update('chunk3')

local final_checksum = h:hexdigest()
print('Adler32:', final_checksum)
```

### 与Zlib结合

```lua
local zlib = require 'zlib'
local adler32 = require 'hashings.adler32'

-- 验证压缩数据
local function verify_zlib_data(compressed, original_adler)
    -- 计算Adler32
    local calc_adler = adler32:new(compressed):hexdigest()

    if calc_adler ~= original_adler then
        return false, 'Adler32 mismatch'
    end

    -- 尝试解压
    local decompressed, err = zlib.decompress(compressed)
    if not decompressed then
        return false, 'Decompression failed: ' .. err
    end

    return decompressed
end
```

## 性能优化

### 大数据处理

```lua
local adler32 = require 'hashings.adler32'

-- Adler32非常快，可以处理大量数据
local function process_large_file(filename)
    local file = io.open(filename, 'rb')
    if not file then return nil end

    local h = adler32:new()
    local total = 0

    while true do
        local block = file:read(1024 * 1024)  -- 1MB块
        if not block then break end
        h:update(block)
        total = total + #block
    end

    file:close()
    print(string.format('Processed %d bytes, Adler32: %s',
        total, h:hexdigest()))
end
```

## 错误处理

```lua
local adler32 = require 'hashings.adler32'

local function safe_checksum(data)
    if type(data) ~= 'string' then
        data = tostring(data)
    end

    local h = adler32:new(data)
    return h:hexdigest()
end

-- 使用
local checksum = safe_checksum(12345)
local checksum = safe_checksum(nil)  -- 转换为"nil"
```
