---

# FCS32校验和 (hashings.fcs32)

FCS32（Frame Check Sequence 32）是32位帧校验序列算法。是FCS16的32位版本，提供更强的错误检测能力。

## 特性

- **输出长度**：32位（4字节）
- **块大小**：8位（逐字节处理）
- **输出格式**：通常表示为8个十六进制字符
- **用途**：数据链路层校验、网络协议
- **特点**：使用查找表，速度快

## 与FCS16比较

| 特性 | FCS16 | FCS32 |
| :--- | :--- | :--- |
| 位宽 | 16位 | 32位 |
| 输出长度 | 2字节 | 4字节 |
| 查找表 | 256项×2字节 | 256项×4字节 |
| 错误检测 | 中等 | 较强 |
| 适用场景 | 简单协议 | 需要更强检测 |

## 使用方法

```lua
local fcs32 = require 'hashings.fcs32'

-- 计算FCS32校验和
local h = fcs32:new('Hello World')
print(h:hexdigest())  -- 输出: 8位十六进制字符串

-- 流式处理
local h = fcs32:new()
h:update('First part ')
h:update('Second part')
print(h:hexdigest())
```

## 接口说明

### new

创建FCS32对象

#### 函数原型

```lua
function fcs32:new(data)
end
```

#### 参数说明

* data
  可选，初始数据字符串

#### 返回值

返回FCS32对象

### update

更新校验和计算

#### 函数原型

```lua
function fcs32:update(data)
end
```

#### 参数说明

* data
  要添加到计算中的数据字符串

### digest

获取二进制校验和

#### 函数原型

```lua
function fcs32:digest()
end
```

#### 返回值

返回4字节的二进制字符串

### hexdigest

获取十六进制校验和

#### 函数原型

```lua
function fcs32:hexdigest()
end
```

#### 返回值

返回8个字符的十六进制字符串（大写）

### copy

复制FCS32对象及其状态

#### 函数原型

```lua
function fcs32:copy()
end
```

#### 返回值

返回包含相同内部状态的新FCS32对象

## 使用示例

### 数据包校验

```lua
local fcs32 = require 'hashings.fcs32'

-- 计算数据包的FCS32
local function add_checksum(data)
    local h = fcs32:new(data)
    local checksum = h:digest()

    -- 数据包格式: [数据][校验和(4)]
    return data .. checksum
end

-- 验证FCS32
local function verify_checksum(packet)
    if #packet < 4 then
        return false, 'Packet too short'
    end

    local data = packet:sub(1, -5)
    local checksum = packet:sub(-4)

    local h = fcs32:new(data)
    local calculated = h:digest()

    return calculated == checksum
end

-- 使用
local data = 'important packet data'
local packet = add_checksum(data)

if verify_checksum(packet) then
    print('Packet valid')
else
    print('Packet corrupted')
end
```

### 网络协议

```lua
local fcs32 = require 'hashings.fcs32'

-- 简单的网络协议
local protocol = {
    VERSION = 0x01,
    TYPE_DATA = 0x01,
    TYPE_ACK = 0x02,
}

local function create_packet(msg_type, data)
    -- 数据包格式: [版本(1)][类型(1)][长度(2)][数据][校验和(4)]
    local header = string.char(protocol.VERSION, msg_type)
    local len = string.pack('<I', #data)

    local packet_no_checksum = header .. len .. data
    local h = fcs32:new(packet_no_checksum)
    local checksum = h:digest()

    return packet_no_checksum .. checksum
end

local function parse_packet(packet)
    if #packet < 8 then
        return nil, 'Packet too short'
    end

    -- 提取头部
    local version = string.byte(packet, 1)
    local msg_type = string.byte(packet, 2)
    local data_len = string.unpack('<I', packet:sub(3, 6))

    if #packet < 8 + data_len then
        return nil, 'Packet incomplete'
    end

    -- 提取数据和校验和
    local data = packet:sub(7, 6 + data_len)
    local checksum_recv = packet:sub(7 + data_len, 10 + data_len)

    -- 验证校验和
    local packet_no_checksum = packet:sub(1, 6 + data_len)
    local h = fcs32:new(packet_no_checksum)
    local checksum_calc = h:digest()

    if checksum_calc ~= checksum_recv then
        return nil, 'Checksum mismatch'
    end

    return {
        version = version,
        msg_type = msg_type,
        data = data
    }
end
```

### 文件校验

```lua
local fcs32 = require 'hashings.fcs32'

local function file_checksum(filename)
    local file = io.open(filename, 'rb')
    if not file then
        return nil, 'Cannot open file'
    end

    local h = fcs32:new()
    local total = 0

    while true do
        local block = file:read(64 * 1024)  -- 64KB块
        if not block then break end
        h:update(block)
        total = total + #block
    end

    file:close()

    return h:hexdigest(), total
end

local checksum, size = file_checksum('test.dat')
print(string.format('File: %d bytes, FCS32: %s', size, checksum))
```

## 算法原理

FCS32使用查找表算法：

1. 初始化校验和为0xFFFFFFFF
2. 对每个字节：
   - 计算索引：`(checksum XOR byte) & 0xFF`
   - 查表并更新：`checksum = (checksum >> 8) XOR table[index]`
3. 返回按位取反的校验和

### 查找表

FCS32使用256项查找表，每项4字节。该表基于多项式：
```
x^32 + x^26 + x^23 + x^22 + x^16 + x^12 + x^11 + x^10 + x^8 + x^7 + x^5 + x^4 + x^2 + x + 1
```

与IEEE 802.3标准的CRC32多项式相同。

## 性能特点

- **计算速度**：快（查找表优化）
- **内存占用**：低（4字节状态 + 1024字节查找表）
- **适用场景**：网络协议、文件校验

## 类属性

```lua
fcs32.digest_size  -- 4 (字节)
fcs32.block_size   -- 4 (字节)
```

## 与其他校验和比较

| 算法 | 长度 | 速度 | 错误检测 | 用途 |
| :--- | :--- | :--- | :--- | :--- |
| FCS16 | 16位 | 快 | 中等 | 串口通信 |
| **FCS32** | **32位** | **快** | **较强** | **网络协议** |
| CRC32 | 32位 | 快 | 较强 | 以太网、ZIP |
| Adler32 | 32位 | 极快 | 较弱 | Zlib |

## 使用场景

### 适合使用

✅ **推荐**：
- 网络协议校验
- 数据链路层帧校验
- 文件完整性验证
- 数据包传输
- 需要比16位更强检测的场景

### 不适合使用

❌ **不推荐**：
- 需要极高安全性（用加密哈希）
- 已经有标准CRC32实现的场景（复用现有代码）
- 资源极度受限的嵌入式系统（用FCS16）

## 与CRC32关系

FCS32实际上与标准CRC32使用相同的多项式和算法，但实现可能略有不同：

```lua
local fcs32 = require 'hashings.fcs32'
local crc32 = require 'hashings.crc32'

local data = 'test data'

local fcs_result = fcs32:new(data):hexdigest()
local crc_result = crc32:new(data):hexdigest()

-- 通常相同或仅字节序不同
print('FCS32:', fcs_result)
print('CRC32:', crc_result)
```

## 完整示例

### 可靠数据传输

```lua
local fcs32 = require 'hashings.fcs32'

-- 发送方
local function send_reliable(socket, data)
    -- 添加校验和
    local h = fcs32:new(data)
    local checksum = h:digest()

    local packet = data .. checksum
    socket:send(packet)

    print(string.format('Sent: %d bytes + 4 bytes checksum', #data))
end

-- 接收方
local function receive_reliable(socket)
    local buffer = ''

    while true do
        local chunk = socket:receive(1024)
        if not chunk then break end

        buffer = buffer .. chunk

        -- 检查是否有完整数据包
        -- 假设数据包以换行符结束
        local newline_pos = buffer:find('\n')
        if newline_pos and newline_pos > 4 then
            -- 提取数据包
            local packet = buffer:sub(1, newline_pos - 1)
            buffer = buffer:sub(newline_pos + 1)

            -- 验证校验和
            if #packet >= 4 then
                local data = packet:sub(1, -5)
                local checksum_recv = packet:sub(-4)

                local h = fcs32:new(data)
                local checksum_calc = h:digest()

                if checksum_calc == checksum_recv then
                    return data
                else
                    print('Checksum error, requesting retransmission')
                    -- 请求重传...
                end
            end
        end
    end

    return nil, 'Receive failed'
end
```

### 批量验证

```lua
local fcs32 = require 'hashings.fcs32'

local function batch_verify(files)
    local results = {}

    for _, filename in ipairs(files) do
        local h = fcs32:new()
        local file = io.open(filename, 'rb')

        if file then
            while true do
                local block = file:read(8192)
                if not block then break end
                h:update(block)
            end

            file:close()
            results[filename] = h:hexdigest()
        else
            results[filename] = nil
        end
    end

    return results
end

local files = {'file1.dat', 'file2.dat', 'file3.dat'}
local checksums = batch_verify(files)

for file, checksum in pairs(checksums) do
    print(string.format('%s: %s', file, checksum))
end
```

## 调试

### 可视化FCS32计算

```lua
local fcs32 = require 'hashings.fcs32'

local function show_fcs32_progress(data)
    local h = fcs32:new()

    for i = 1, #data do
        local byte = string.byte(data, i)
        local before = h._crc:asnumber()

        h:update(string.char(byte))

        local after = h._crc:asnumber()
        print(string.format('Byte: 0x%02X, CRC before: 0x%08X, CRC after: 0x%08X',
            byte, before, after))
    end

    print('Final FCS32:', h:hexdigest())
end

show_fcs32_progress('ABCD')
```

## 注意事项

⚠️ **重要**：

1. **不是加密**：FCS32是校验和，不是加密哈希
2. **碰撞可能**：32位存在碰撞可能性
3. **字节序**：注意大端序和小端序的转换
4. **标准差异**：与标准CRC32可能略有差异
5. **错误检测**：适合一般错误检测，不适合防篡改

## 最佳实践

1. **协议设计**：在自定义协议中使用FCS32进行错误检测
2. **文件校验**：用于快速文件完整性验证
3. **错误处理**：总是检查校验和并处理错误
4. **性能考虑**：FCS32很快，适合实时处理
5. **测试验证**：在实际环境中测试误报率和漏报率
