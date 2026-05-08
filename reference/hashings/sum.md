---

# Sum校验和 (hashings.sum)

Sum是最简单的8位校验和算法，通过累加所有字节的值来计算校验和。这是最基本的校验和形式，计算速度极快但错误检测能力很弱。

## 特性

- **输出长度**：8位（1字节）
- **块大小**：8位（逐字节处理）
- **输出格式**：通常表示为2个十六进制字符
- **用途**：最简单的数据校验
- **特点**：计算极快，检测能力弱

## 与其他简单校验和比较

| 特性 | Sum | Sum16 | Adler32 |
| :--- | :--- | :--- | :--- |
| 位宽 | 8位 | 16位 | 32位 |
| 算法 | 简单累加 | 累加+折叠 | 双累加器 |
| 速度 | 极快 | 极快 | 快 |
| 错误检测 | 很弱 | 弱 | 中等 |
| 复杂度 | 最低 | 低 | 中 |
| 输出长度 | 2字符十六进制 | 4字符十六进制 | 8字符十六进制 |

## 使用方法

```lua
local sum = require 'hashings.sum'

-- 计算Sum校验和
local h = sum:new('Hello World')
print(h:hexdigest())  -- 输出: 2位十六进制字符串

-- 流式处理
local h = sum:new()
h:update('First part ')
h:update('Second part')
print(h:hexdigest())
```

## 接口说明

### new

创建Sum对象

#### 函数原型

```lua
function sum:new(data)
end
```

#### 参数说明

* data
  可选，初始数据字符串

#### 返回值

返回Sum对象

### update

更新校验和计算

#### 函数原型

```lua
function sum:update(data)
end
```

#### 参数说明

* data
  要添加到计算中的数据字符串

### digest

获取二进制校验和

#### 函数原型

```lua
function sum:digest()
end
```

#### 返回值

返回1字节的二进制字符串

### hexdigest

获取十六进制校验和

#### 函数原型

```lua
function sum:hexdigest()
end
```

#### 返回值

返回2个字符的十六进制字符串（大写）

### copy

复制Sum对象及其状态

#### 函数原型

```lua
function sum:copy()
end
```

#### 返回值

返回包含相同内部状态的新Sum对象

## 使用示例

### 基础校验

```lua
local sum = require 'hashings.sum'

-- 计算简单的字节校验和
local function simple_checksum(data)
    local h = sum:new(data)
    return h:hexdigest()
end

-- 验证数据
local function verify_simple(data, expected)
    local calculated = simple_checksum(data)
    return calculated == expected
end

local data = 'Hello'
local checksum = simple_checksum(data)

print('Data:', data)
print('Checksum:', checksum)

if verify_simple(data, checksum) then
    print('Verification passed')
end
```

### 快速数据检查

```lua
local sum = require 'hashings.sum'

-- 快速检查数据是否被修改
local function quick_check(data)
    local h = sum:new(data)
    return h:hexdigest()
end

-- 存储数据和校验和
local data_store = {}
local function store_data(key, value)
    data_store[key] = {
        data = value,
        checksum = quick_check(value)
    }
end

local function retrieve_data(key)
    local record = data_store[key]
    if not record then
        return nil, 'Not found'
    end

    -- 验证校验和
    local current_checksum = quick_check(record.data)
    if current_checksum ~= record.checksum then
        return nil, 'Data corrupted'
    end

    return record.data
end

-- 使用
store_data('user1', 'Alice')
store_data('user2', 'Bob')

local data, err = retrieve_data('user1')
if data then
    print('User1:', data)
else
    print('Error:', err)
end
```

### 简单协议

```lua
local sum = require 'hashings.sum'

-- 极简单的命令协议
local function create_command(cmd, payload)
    -- 格式: [CMD(1)][长度(1)][数据(N)][校验和(1)]
    local cmd_byte = string.char(cmd)
    local len_byte = string.char(#payload & 0xFF)

    local packet_without_checksum = cmd_byte .. len_byte .. payload
    local h = sum:new(packet_without_checksum)
    local checksum = h:digest()

    return packet_without_checksum .. checksum
end

local function parse_command(packet)
    if #packet < 3 then
        return nil, 'Packet too short'
    end

    local cmd = string.byte(packet, 1)
    local len = string.byte(packet, 2)

    if #packet < 3 + len then
        return nil, 'Packet incomplete'
    end

    local payload = packet:sub(3, 2 + len)
    local checksum_recv = packet:sub(3 + len)

    -- 验证校验和
    local data_part = packet:sub(1, 2 + len)
    local h = sum:new(data_part)
    local checksum_calc = h:digest()

    if checksum_calc ~= checksum_recv then
        return nil, 'Checksum error'
    end

    return {
        cmd = cmd,
        payload = payload
    }
end

-- 使用
local CMD_ECHO = 0x01
local packet = create_command(CMD_ECHO, 'Hello')

local parsed, err = parse_command(packet)
if parsed then
    print('Command:', parsed.cmd)
    print('Payload:', parsed.payload)
else
    print('Error:', err)
end
```

### 内存校验

```lua
local sum = require 'hashings.sum'

-- 校验数据块
local function checksum_block(block)
    local h = sum:new()
    h:update(block)
    return h:hexdigest()
end

-- 分块校验大文件
local function checksum_chunks(filename, chunk_size)
    chunk_size = chunk_size or 4096

    local file = io.open(filename, 'rb')
    if not file then
        return nil, 'Cannot open file'
    end

    local checksums = {}
    local chunk_num = 0

    while true do
        local chunk = file:read(chunk_size)
        if not chunk then break end

        chunk_num = chunk_num + 1
        local checksum = checksum_block(chunk)
        table.insert(checksums, {
            number = chunk_num,
            checksum = checksum,
            size = #chunk
        })
    end

    file:close()
    return checksums
end

-- 使用
local checksums, err = checksum_chunks('test.dat')
if checksums then
    for _, cs in ipairs(checksums) do
        print(string.format('Chunk %d: %s (%d bytes)',
            cs.number, cs.checksum, cs.size))
    end
else
    print('Error:', err)
end
```

## 算法原理

Sum算法是最简单的校验和：

1. 初始化累加器为0
2. 对每个字节：
   - 累加字节值到累加器
3. 返回累加器的值

### 示例计算

```
数据: "ABC"

字节值: A=65, B=66, C=67

计算:
sum = 0
sum = 0 + 65 = 65
sum = 65 + 66 = 131
sum = 131 + 67 = 198

最终校验和 = 198 = 0xC6
```

## 性能特点

- **计算速度**：极快（仅加法操作）
- **内存占用**：极低（1字节状态）
- **适用场景**：最简单的数据完整性检查

## 类属性

```lua
sum.digest_size  -- 1 (字节)
sum.block_size   -- 1 (字节)
```

## 局限性

⚠️ **重要限制**：

1. **碰撞率极高**：只有256个可能值
2. **顺序无关**：{A, B}和{B, A}的校验和相同
3. **错误检测弱**：很容易漏检错误
4. **不安全**：完全不可用于安全场景
5. **补码抵消**：互补的字节会相互抵消

### 碰撞示例

```lua
local sum = require 'hashings.sum'

-- 这些不同的数据会产生相同的校验和
local data1 = 'AB'      -- 65 + 66 = 131
local data2 = '\x83'    -- 131
local data3 = 'AC'      -- 65 + 67 = 132 (接近)

print('Sum("AB"):', sum:new(data1):hexdigest())
print('Sum("\\x83"):', sum:new(data2):hexdigest())
```

## 使用场景

### 适合使用

✅ **可以用于**：
- 快速预检
- 非关键数据
- 教学示例
- 最简单的完整性检查
- 与其他校验和组合使用

### 不适合使用

❌ **不可用于**：
- 任何重要数据
- 网络传输
- 文件存储
- 安全相关应用
- 需要可靠错误检测的场景

## 完整示例

### 快速数据验证

```lua
local sum = require 'hashings.sum'

-- 快速检查数据变化
local function detect_change(data, original_checksum)
    local current = sum:new(data):hexdigest()
    return current ~= original_checksum, current
end

-- 监控数据变化
local data_cache = {}

local function update_cache(key, data)
    data_cache[key] = {
        data = data,
        checksum = sum:new(data):hexdigest()
    }
end

local function check_cache(key)
    local record = data_cache[key]
    if not record then
        return false, 'Not in cache'
    end

    local changed, current = detect_change(record.data, record.checksum)
    return changed, current
end

-- 使用
update_cache('config', 'server=127.0.0.1')

-- 检查是否变化
local changed, checksum = check_cache('config')
if not changed then
    print('Config unchanged')
else
    print('Config changed!')
end
```

### 组合校验

```lua
local sum = require 'hashings.sum'
local sum16 = require 'hashings.sum16'

-- 使用多个校验和层次
local function multi_level_checksum(data)
    return {
        sum8 = sum:new(data):hexdigest(),           -- 快速预检
        sum16 = sum16:new(data):hexdigest()         -- 二次验证
    }
end

local function verify_multi_level(data, checksums)
    -- 快速检查
    local calc_sum8 = sum:new(data):hexdigest()
    if calc_sum8 ~= checksums.sum8 then
        return false, 'Level 1 check failed'
    end

    -- 二次检查
    local calc_sum16 = sum16:new(data):hexdigest()
    if calc_sum16 ~= checksums.sum16 then
        return false, 'Level 2 check failed'
    end

    return true
end

-- 使用
local data = 'important data'
local checksums = multi_level_checksum(data)
print('Checksums:', checksums.sum8, checksums.sum16)

local valid, err = verify_multi_level(data, checksums)
if valid then
    print('All checks passed')
end
```

## 调试

### 可视化Sum计算

```lua
local sum = require 'hashings.sum'

local function show_sum_calculation(data)
    local h = sum:new()

    print('Sum Calculation:')
    print('Data:', data)
    print('Length:', #data)
    print()

    for i = 1, #data do
        local byte = string.byte(data, i)
        local before = h._sum:asnumber()

        h:update(string.char(byte))

        local after = h._sum:asnumber()
        print(string.format('  [%d] "%c" (0x%02X = %3d): %d -> %d',
            i, byte, byte, byte, before, after))
    end

    print()
    print('Final sum:', h._sum:asnumber())
    print('Hexdigest:', h:hexdigest())
end

show_sum_calculation('Hello')
```

## 性能测试

```lua
local sum = require 'hashings.sum'
local sum16 = require 'hashings.sum16'
local adler32 = require 'hashings.adler32'

local function benchmark(hash_func, data, iterations)
    local start = os.clock()

    for i = 1, iterations do
        hash_func:new(data):hexdigest()
    end

    return os.clock() - start
end

local data = string.rep('test', 1000)
local iterations = 10000

local sum_time = benchmark(sum, data, iterations)
local sum16_time = benchmark(sum16, data, iterations)
local adler_time = benchmark(adler32, data, iterations)

print(string.format('Sum:     %.4f seconds (%.0f ops/sec)',
    sum_time, iterations / sum_time))
print(string.format('Sum16:   %.4f seconds (%.0f ops/sec)',
    sum16_time, iterations / sum16_time))
print(string.format('Adler32: %.4f seconds (%.0f ops/sec)',
    adler_time, iterations / adler_time))
```

## 注意事项

⚠️ **重要**：

1. **极弱的检测能力**：只能检测最明显的错误
2. **高碰撞率**：只有256个可能值
3. **不安全**：绝不可用于安全相关场景
4. **建议替代**：几乎所有情况下都应该用更强的校验和
5. **谨慎使用**：只在明确理解其限制的情况下使用

## 最佳实践

1. **仅作预检**：用作快速的第一道检查
2. **组合使用**：与其他校验和方法组合
3. **明确文档**：清楚标注使用Sum的局限性
4. **考虑替代**：优先使用Sum16、Adler32或更强的算法
5. **测试充分**：在实际环境中验证是否满足需求

## 迁移建议

### 从Sum迁移到Sum16

```lua
-- 之前（Sum）
local sum = require 'hashings.sum'
local checksum = sum:new(data):hexdigest()  -- 2字符

-- 之后（Sum16）
local sum16 = require 'hashings.sum16'
local checksum = sum16:new(data):hexdigest()  -- 4字符
-- 错误检测能力更强，碰撞率大幅降低
```

### 从Sum迁移到Adler32

```lua
-- 之前（Sum）
local sum = require 'hashings.sum'
local checksum = sum:new(data):hexdigest()

-- 之后（Adler32）
local adler32 = require 'hashings.adler32'
local checksum = adler32:new(data):hexdigest()
-- 32位校验和，错误检测能力强得多
```
