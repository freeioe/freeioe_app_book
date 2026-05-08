---

# FCS16校验和 (hashings.fcs16)

FCS16（Frame Check Sequence 16）是16位帧校验序列算法，定义在RFC 1331标准中。用于数据链路层的错误检测。

## 特性

- **输出长度**：16位（2字节）
- **块大小**：8位（逐字节处理）
- **输出格式**：通常表示为4个十六进制字符
- **用途**：串行通信、数据链路层校验
- **标准**：RFC 1331

## 使用方法

```lua
local fcs16 = require 'hashings.fcs16'

-- 计算FCS16校验和
local h = fcs16:new('Hello World')
print(h:hexdigest())  -- 输出: 4位十六进制字符串

-- 流式处理
local h = fcs16:new()
h:update('First part ')
h:update('Second part')
print(h:hexdigest())
```

## 接口说明

### new

创建FCS16对象

#### 函数原型

```lua
function fcs16:new(data)
end
```

#### 参数说明

* data
  可选，初始数据字符串

#### 返回值

返回FCS16对象

### update

更新校验和计算

#### 函数原型

```lua
function fcs16:update(data)
end
```

#### 参数说明

* data
  要添加到计算中的数据字符串

### digest

获取二进制校验和

#### 函数原型

```lua
function fcs16:digest()
end
```

#### 返回值

返回2字节的二进制字符串

### hexdigest

获取十六进制校验和

#### 函数原型

```lua
function fcs16:hexdigest()
end
```

#### 返回值

返回4个字符的十六进制字符串（大写）

### copy

复制FCS16对象及其状态

#### 函数原型

```lua
function fcs16:copy()
end
```

#### 返回值

返回包含相同内部状态的新FCS16对象

## 使用示例

### 串口数据校验

```lua
local fcs16 = require 'hashings.fcs16'

-- 计算数据包的FCS16
local function add_fcs16(data)
    local h = fcs16:new(data)
    local checksum = h:digest()

    return data .. checksum
end

-- 验证FCS16
local function verify_fcs16(packet)
    if #packet < 2 then
        return false, 'Packet too short'
    end

    local data = packet:sub(1, -3)
    local checksum = packet:sub(-2)

    local h = fcs16:new(data)
    local calculated = h:digest()

    return calculated == checksum
end
```

### HDLC协议

```lua
local fcs16 = require 'hashings.fcs16'

-- HDLC帧格式: [标志][地址][控制][数据][FCS][标志]
local function create_hdlc_frame(addr, ctrl, data)
    local frame = string.char(0xFF) .. addr .. ctrl .. data

    local h = fcs16:new(frame)
    local fcs = h:digest()

    return frame .. fcs .. string.char(0x7E)
end

local function parse_hdlc_frame(frame)
    -- 查找标志
    local start_pos = frame:find(string.char(0x7E))
    local end_pos = frame:find(string.char(0x7E), 2)

    if not start_pos or not end_pos then
        return nil, 'Invalid frame'
    end

    local data_part = frame:sub(start_pos + 1, end_pos - 1)

    -- 验证FCS
    local payload = data_part:sub(1, -3)
    local fcs_received = data_part:sub(-2)

    local h = fcs16:new(payload)
    local fcs_calc = h:digest()

    if fcs_calc ~= fcs_received then
        return nil, 'FCS mismatch'
    end

    return payload:sub(1, -3)  -- 去除FCS的数据
end
```

### 数据链路层

```lua
local fcs16 = require 'hashings.fcs16'

-- 串口通信
local function send_serial_data(serial, data)
    local packet = data .. fcs16:new(data):digest()

    serial:send(packet)
    print('Sent:', #packet, 'bytes with FCS16')
end

local function receive_serial_data(serial)
    local buffer = ''

    while true do
        local byte = serial:receive(1)
        buffer = buffer .. byte

        -- 检查是否收到完整数据包
        -- 假设数据包以特定标记结束
        if byte == 0x0A then  -- 换行符
            -- 验证FCS
            if #buffer >= 2 then
                local data = buffer:sub(1, -3)
                local fcs_recv = buffer:sub(-2)

                local h = fcs16:new(data)
                local fcs_calc = h:digest()

                if fcs_calc == fcs_recv then
                    return data
                else
                    print('FCS error, discarding packet')
                end
            end

            buffer = ''
        end
    end
end
```

## 算法原理

FCS16使用查找表算法：

1. 初始化校验和为0xFFFF
2. 对每个字节：
   - 计算索引：`(checksum XOR byte) & 0xFF`
   - 查表并更新：`checksum = (checksum >> 8) XOR table[index]`
3. 返回按位取反的校验和

### 查找表

FCS16使用256项查找表，该表基于多项式：
```
x^16 + x^12 + x^5 + 1
```

## 性能特点

- **计算速度**：快（查找表优化）
- **内存占用**：极低（2字节状态 + 512字节查找表）
- **适用场景**：串行通信、数据链路层

## 类属性

```lua
fcs16.digest_size  -- 2 (字节)
fcs16.block_size   -- 8 (位，逐字节处理)
```

## 与其他校验和比较

| 算法 | 长度 | 速度 | 错误检测 | 用途 |
| :--- | :--- | :--- | :--- | :--- |
| **FCS16** | **16位** | **快** | **中等** | **串口通信** |
| CRC16-CCITT | 16位 | 快 | 中等 | X.25等 |
| CRC16-Modbus | 16位 | 快 | 中等 | Modbus |
| Adler32 | 32位 | 极快 | 较弱 | Zlib |
| CRC32 | 32位 | 快 | 好 | 以太网 |

## 使用场景

### 适合使用

✅ **推荐**：
- 串行通信协议（RS-232, RS-485）
- HDLC协议
- 数据链路层帧校验
- 嵌入式通信
- 简单的包校验

### 不适合使用

❌ **不推荐**：
- 需要强错误检测（用CRC32）
- 大规模数据传输
- 文件存储校验
- 网络协议（用更强大的校验）

## 实际应用

### Modbus RTU

```lua
local fcs16 = require 'hashings.fcs16'

-- Modbus RTU使用CRC16，不是FCS16
-- 但某些变体可能使用FCS16
local function add_checksum(data)
    local h = fcs16:new(data)
    return data .. h:digest()
end
```

### 自定义协议

```lua
local fcs16 = require 'hashings.fcs16'

local protocol = {
    START = 0x02,
    END = 0x03,
    ESC = 0x1B
}

local function encode_packet(payload)
    local packet = string.char(protocol.START)
    packet = packet .. payload

    local h = fcs16:new(packet)
    local fcs = h:digest()

    packet = packet .. fcs
    packet = packet .. string.char(protocol.END)

    return packet
end

local function decode_packet(packet)
    -- 移除首尾标记
    local data = packet:sub(2, -2)

    if #data < 2 then
        return nil, 'Packet too short'
    end

    -- 分离数据和FCS
    local payload = data:sub(1, -3)
    local fcs_received = data:sub(-2)

    -- 验证FCS
    local h = fcs16:new(payload)
    local fcs_calc = h:digest()

    if fcs_calc ~= fcs_received then
        return nil, 'FCS checksum error'
    end

    return payload
end
```

### 错误处理

```lua
local fcs16 = require 'hashings.fcs16'

local function safe_verify(packet)
    if type(packet) ~= 'string' then
        return false, 'Invalid packet type'
    end

    if #packet < 2 then
        return false, 'Packet too short'
    end

    local data = packet:sub(1, -3)
    local checksum = packet:sub(-2)

    local h = fcs16:new(data)
    local calculated = h:digest()

    if calculated ~= checksum then
        -- FCS错误
        return false, 'Checksum mismatch'
    end

    return true
end

-- 使用
local data = 'test data'
local packet = data .. fcs16:new(data):digest()

if safe_verify(packet) then
    print('Packet valid')
else
    print('Packet error')
end
```

## 注意事项

⚠️ **重要**：

1. **标准差异**：FCS16不同于CRC16，查找表和多项式都不同
2. **协议匹配**：确认目标协议使用FCS16而非CRC16
3. **字节序**：FCS16大端序，某些平台可能需要转换
4. **错误检测**：16位校验和相对较弱，不适合大块数据

## 调试

### 可视化FCS计算

```lua
local fcs16 = require 'hashings.fcs16'

local function show_fcs_progress(data)
    local h = fcs16:new()

    for i = 1, #data do
        local byte = string.byte(data, i)
        local before = h._crc:asnumber()

        h:update(string.char(byte))

        local after = h._crc:asnumber()
        print(string.format('Byte: 0x%02X, CRC before: 0x%04X, CRC after: 0x%04X',
            byte, before, after))
    end

    print('Final FCS16:', h:hexdigest())
end

show_fcs_progress('ABC')
```

## 与CRC16互操作

如果需要同时支持多种校验和：

```lua
local fcs16 = require 'hashings.fcs16'
local crc16 = require 'hashings.crc16'  -- 假设存在

local function calculate_checksums(data, algorithm)
    if algorithm == 'FCS16' then
        return fcs16:new(data):digest()
    elseif algorithm == 'CRC16' then
        return crc16:new(data):digest()
    else
        error('Unsupported algorithm')
    end
end
```

## 性能优化

### 批量处理

```lua
local fcs16 = require 'hashings.fcs16'

local function batch_checksum(packets)
    local results = {}

    for i, packet in ipairs(packets) do
        local h = fcs16:new(packet)
        results[i] = h:hexdigest()
    end

    return results
end
```

### 内存高效处理

```lua
local fcs16 = require 'hashings.fcs16'

-- 复用对象减少内存分配
local h = fcs16:new()

for i = 1, 100 do
    h:reset()  -- 如果有reset方法
    -- 或者
    h = fcs16:new()  -- 重新创建

    local data = generate_data(i)
    h:update(data)

    local checksum = h:hexdigest()
    process(data, checksum)
end
```

## 最佳实践

1. **协议匹配**：确认使用FCS16而非CRC16
2. **错误处理**：检查返回值并处理错误
3. **性能考虑**：FCS16很快，但查找表占用512字节
4. **测试验证**：在实际环境中测试校验效果
5. **文档记录**：明确标注使用的FCS16标准
