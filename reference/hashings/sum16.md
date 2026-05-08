---

# Sum16校验和 (hashings.sum16)

Sum16是一种简单的16位校验和算法，通过累加16位值并折叠进位来计算校验和。常用于简单协议和基础数据验证。

## 特性

- **输出长度**：16位（2字节）
- **块大小**：16位（逐16位处理）
- **输出格式**：通常表示为4个十六进制字符
- **用途**：简单数据校验、基础协议
- **特点**：计算简单，速度极快

## 与其他简单校验和比较

| 特性 | Sum | Sum16 | FCS16 |
| :--- | :--- | :--- | :--- |
| 位宽 | 8位 | 16位 | 16位 |
| 算法 | 简单累加 | 累加+折叠 | CRC查找表 |
| 速度 | 极快 | 极快 | 快 |
| 错误检测 | 弱 | 中等 | 中等 |
| 复杂度 | 最低 | 低 | 中 |

## 使用方法

```lua
local sum16 = require 'hashings.sum16'

-- 计算Sum16校验和
local h = sum16:new('Hello World')
print(h:hexdigest())  -- 输出: 4位十六进制字符串

-- 流式处理
local h = sum16:new()
h:update('First part ')
h:update('Second part')
print(h:hexdigest())
```

## 接口说明

### new

创建Sum16对象

#### 函数原型

```lua
function sum16:new(data)
end
```

#### 参数说明

* data
  可选，初始数据字符串

#### 返回值

返回Sum16对象

### update

更新校验和计算

#### 函数原型

```lua
function sum16:update(data)
end
```

#### 参数说明

* data
  要添加到计算中的数据字符串

#### 注意

如果数据长度为奇数，会自动补零。

### digest

获取二进制校验和

#### 函数原型

```lua
function sum16:digest()
end
```

#### 返回值

返回2字节的二进制字符串

### hexdigest

获取十六进制校验和

#### 函数原型

```lua
function sum16:hexdigest()
end
```

#### 返回值

返回4个字符的十六进制字符串（大写）

### copy

复制Sum16对象及其状态

#### 函数原型

```lua
function sum16:copy()
end
```

#### 返回值

返回包含相同内部状态的新Sum16对象

## 使用示例

### 基础数据校验

```lua
local sum16 = require 'hashings.sum16'

-- 计算数据的Sum16校验和
local function calculate_checksum(data)
    local h = sum16:new(data)
    return h:hexdigest()
end

-- 验证数据
local function verify_data(data, expected_checksum)
    local calculated = calculate_checksum(data)
    return calculated == expected_checksum
end

local data = 'test data packet'
local checksum = calculate_checksum(data)

print('Data:', data)
print('Checksum:', checksum)

if verify_data(data, checksum) then
    print('Verification passed')
end
```

### 简单协议

```lua
local sum16 = require 'hashings.sum16'

-- 简单的数据包协议
local function create_packet(data)
    -- 数据包格式: [长度(2)][数据][校验和(2)]
    local len = string.pack('<I', #data)

    local packet_without_checksum = len .. data
    local h = sum16:new(packet_without_checksum)
    local checksum = h:digest()

    return packet_without_checksum .. checksum
end

local function parse_packet(packet)
    if #packet < 4 then
        return nil, 'Packet too short'
    end

    -- 读取长度
    local data_len = string.unpack('<I', packet:sub(1, 2))

    if #packet < 4 + data_len then
        return nil, 'Packet incomplete'
    end

    -- 提取数据部分（包括长度字段）
    local data_with_len = packet:sub(1, 2 + data_len)
    local checksum_recv = packet:sub(3 + data_len, 4 + data_len)

    -- 验证校验和
    local h = sum16:new(data_with_len)
    local checksum_calc = h:digest()

    if checksum_calc ~= checksum_recv then
        return nil, 'Checksum mismatch'
    end

    -- 返回实际数据
    return packet:sub(3, 2 + data_len)
end

-- 使用示例
local data = 'Hello, World!'
local packet = create_packet(data)
print('Packet length:', #packet)

local parsed, err = parse_packet(packet)
if parsed then
    print('Parsed data:', parsed)
else
    print('Error:', err)
end
```

### 二进制数据

```lua
local sum16 = require 'hashings.sum16'

-- 处理二进制数据
local function create_binary_packet(values)
    -- 将数值数组转换为二进制数据
    local data = ''
    for _, v in ipairs(values) do
        data = data .. string.pack('<I', v)
    end

    -- 计算校验和
    local h = sum16:new(data)
    local checksum = h:digest()

    return data .. checksum
end

local function parse_binary_packet(packet)
    if #packet < 4 or (#packet % 2) ~= 0 then
        return nil, 'Invalid packet'
    end

    -- 移除校验和
    local data = packet:sub(1, -3)
    local checksum_recv = packet:sub(-2)

    -- 验证
    local h = sum16:new(data)
    local checksum_calc = h:digest()

    if checksum_calc ~= checksum_recv then
        return nil, 'Checksum error'
    end

    -- 解析数据
    local values = {}
    for i = 1, #data, 4 do
        local val = string.unpack('<I', data:sub(i, i+3))
        table.insert(values, val)
    end

    return values
end

-- 使用
local values = {0x1234, 0x5678, 0x9ABC, 0xDEF0}
local packet = create_binary_packet(values)

local parsed, err = parse_binary_packet(packet)
if parsed then
    print('Parsed values:', table.concat(parsed, ', '))
else
    print('Error:', err)
end
```

## 算法原理

Sum16算法非常简单：

1. 将数据按16位分组
2. 累加所有16位值到32位累加器
3. 将累加器的进位折叠回低16位
4. 返回按位取反的结果

### 示例计算

```
数据: 0x1234 0x5678 0x9ABC

步骤:
1. sum = 0
2. sum = 0 + 0x1234 = 0x1234
3. sum = 0x1234 + 0x5678 = 0x68AC
4. sum = 0x68AC + 0x9ABC = 0x10368

进位折叠:
- sum = 0x0368 + 0x0001 = 0x0369

取反:
- result = ~0x0369 = 0xFC96
```

## 性能特点

- **计算速度**：极快（简单加法）
- **内存占用**：极低（4字节状态）
- **适用场景**：简单协议、快速验证

## 类属性

```lua
sum16.digest_size  -- 2 (字节)
sum16.block_size   -- 2 (字节)
```

## 使用场景

### 适合使用

✅ **推荐**：
- 简单协议校验
- 快速数据验证
- 资源受限系统
- 非关键数据完整性检查
- 教学示例

### 不适合使用

❌ **不推荐**：
- 需要强错误检测（用CRC或加密哈希）
- 网络传输（用更强大的校验和）
- 安全敏感应用
- 重要文件存储

## 局限性

⚠️ **重要限制**：

1. **错误检测弱**：简单的累加容易产生碰撞
2. **顺序敏感**：数据顺序改变可能产生相同校验和
3. **不安全**：完全不适合安全相关应用
4. **容易伪造**：攻击者可以轻松构造具有相同校验和的数据

## 完整示例

### 传感器数据校验

```lua
local sum16 = require 'hashings.sum16'

-- 模拟传感器数据包
local function create_sensor_packet(sensor_id, timestamp, values)
    -- 数据包格式: [ID(4)][时间(4)][值数量(2)][值1(4)][值2(4)]...[校验和(2)]
    local packet = string.pack('<II', sensor_id, timestamp)
    packet = packet .. string.pack('<I', #values)

    for _, v in ipairs(values) do
        packet = packet .. string.pack('<i', v)  -- 有符号整数
    end

    -- 计算校验和
    local h = sum16:new(packet)
    local checksum = h:digest()

    return packet .. checksum
end

local function parse_sensor_packet(packet)
    if #packet < 12 then
        return nil, 'Packet too short'
    end

    -- 提取数据部分
    local data_part = packet:sub(1, -3)
    local checksum_recv = packet:sub(-2)

    -- 验证校验和
    local h = sum16:new(data_part)
    local checksum_calc = h:digest()

    if checksum_calc ~= checksum_recv then
        return nil, 'Checksum mismatch'
    end

    -- 解析数据
    local sensor_id, timestamp = string.unpack('<II', data_part:sub(1, 8))
    local value_count = string.unpack('<I', data_part:sub(9, 12))

    local values = {}
    local offset = 13
    for i = 1, value_count do
        if offset + 3 > #data_part then
            break
        end
        local val = string.unpack('<i', data_part:sub(offset, offset + 3))
        table.insert(values, val)
        offset = offset + 4
    end

    return {
        sensor_id = sensor_id,
        timestamp = timestamp,
        values = values
    }
end

-- 使用示例
local sensor_id = 0x12345678
local timestamp = os.time()
local values = {25, 26, 24, 27, 25}

local packet = create_sensor_packet(sensor_id, timestamp, values)
print('Packet length:', #packet)

local parsed, err = parse_sensor_packet(packet)
if parsed then
    print('Sensor ID:', string.format('0x%08X', parsed.sensor_id))
    print('Timestamp:', parsed.timestamp)
    print('Values:', table.concat(parsed.values, ', '))
else
    print('Error:', err)
end
```

### 内存缓冲区校验

```lua
local sum16 = require 'hashings.sum16'

-- 校验内存缓冲区
local function checksum_buffer(buffer)
    local h = sum16:new()

    -- 假设buffer是字符串形式的数据
    h:update(buffer)

    return h:hexdigest()
end

-- 分段处理大缓冲区
local function checksum_large_buffer(buffer, chunk_size)
    local h = sum16:new()
    chunk_size = chunk_size or 4096

    for i = 1, #buffer, chunk_size do
        local chunk = buffer:sub(i, i + chunk_size - 1)
        h:update(chunk)
    end

    return h:hexdigest()
end

-- 比较两个缓冲区
local function compare_buffers(buf1, buf2)
    local sum1 = checksum_buffer(buf1)
    local sum2 = checksum_buffer(buf2)

    return sum1 == sum2, sum1, sum2
end

-- 使用
local buffer1 = string.rep('ABCD', 1000)
local buffer2 = string.rep('ABCD', 1000)
local buffer3 = string.rep('ABCE', 1000)

local same, s1, s2 = compare_buffers(buffer1, buffer2)
print('Buffer1 == Buffer2:', same)

local same, s1, s3 = compare_buffers(buffer1, buffer3)
print('Buffer1 == Buffer3:', same)
```

## 调试

### 可视化Sum16计算

```lua
local sum16 = require 'hashings.sum16'

local function show_sum16_calculation(data)
    -- 确保数据长度为偶数
    if (#data % 2) == 1 then
        data = data .. string.char(0)
    end

    local h = sum16:new()

    print('Sum16 Calculation:')
    print('Data length:', #data)

    for i = 1, #data, 2 do
        local byte1 = string.byte(data, i)
        local byte2 = string.byte(data, i + 1)
        local value = byte1 * 256 + byte2

        local before = h._sum:asnumber()
        h:update(data:sub(i, i + 1))
        local after = h._sum:asnumber()

        print(string.format('  [%d] 0x%02X%02X (%5d): 0x%08X -> 0x%08X',
            i // 2, byte1, byte2, value, before, after))
    end

    print('Final checksum (before inversion):', string.format('0x%04X', h._sum:asnumber()))
    print('Final Sum16:', h:hexdigest())
end

show_sum16_calculation('ABCD')
```

## 性能测试

```lua
local sum16 = require 'hashings.sum16'
local fcs16 = require 'hashings.fcs16'

local function benchmark(hash_func, data, iterations)
    local start = os.clock()

    for i = 1, iterations do
        hash_func:new(data):hexdigest()
    end

    return os.clock() - start
end

local data = string.rep('test data', 1000)
local iterations = 10000

local sum16_time = benchmark(sum16, data, iterations)
local fcs16_time = benchmark(fcs16, data, iterations)

print(string.format('Sum16:  %.4f seconds (%.0f ops/sec)',
    sum16_time, iterations / sum16_time))
print(string.format('FCS16:  %.4f seconds (%.0f ops/sec)',
    fcs16_time, iterations / fcs16_time))
print(string.format('Speedup: %.2fx', fcs16_time / sum16_time))
```

## 注意事项

⚠️ **重要**：

1. **弱校验和**：只适合最简单的错误检测
2. **自动补零**：奇数长度数据会自动补零
3. **字节序**：使用大端序（网络字节序）
4. **碰撞率高**：不适合重要数据
5. **不安全**：绝不可用于安全相关应用

## 最佳实践

1. **仅用于简单场景**：只在非关键应用中使用
2. **考虑替代方案**：优先使用CRC或更强的校验和
3. **组合使用**：可以与其他校验和方法组合
4. **明确文档**：在使用Sum16的地方明确标注其局限性
5. **测试充分**：在实际环境中验证错误检测能力
