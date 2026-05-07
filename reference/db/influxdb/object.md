---

# InfluxDB对象模块 (db.influxdb.object)

本模块提供面向对象的InfluxDB写入接口，支持灵活的数据构建和批量写入。

## 功能说明

对象模块提供了一个面向对象的接口来构建和写入InfluxDB数据。主要特性：

- **灵活的数据构建**：逐步添加标签和字段
- **批量写入支持**：支持缓冲多个数据点后批量写入
- **立即写入**：也可以立即写入单个数据点
- **状态管理**：自动管理时间戳和数据状态

## 使用方法

```lua
local object = require 'db.influxdb.object'

-- 创建InfluxDB对象
local obj = object:new({
    proto = 'http',
    host = 'localhost',
    port = 8086,
    db = 'mydb',
    precision = 'ms'
})

-- 设置测量名称
obj:set_measurement('temperature')

-- 添加标签
obj:add_tag('location', 'room1')
obj:add_tag('sensor', 'S001')

-- 添加字段
obj:add_field('value', 25.5)

-- 立即写入
obj:write()
```

## 接口说明

### new

创建新的InfluxDB对象

#### 函数原型

```lua
function object:new(opts)
end
```

#### 参数说明

* opts
  配置选项表

| 选项 | 类型 | 默认值 | 说明 |
| :--- | :--- | :--- | :--- |
| proto | string | 'http' | 写入协议（http/udp） |
| host | string | '127.0.0.1' | InfluxDB主机地址 |
| port | number | 8086 | InfluxDB端口 |
| db | string | 'influx' | 数据库名称 |
| hostname | string | 同host | 主机名 |
| precision | string | 'ms' | 时间精度（s/ms） |
| ssl | boolean | false | 是否使用SSL/TLS |
| auth | string | nil | 认证信息（username:password） |

#### 返回值

成功返回InfluxDB对象，失败返回nil和错误信息

#### 示例

```lua
local obj = object:new({
    proto = 'http',
    host = 'localhost',
    port = 8086,
    db = 'sensors',
    precision = 'ms'
})

-- 带认证的对象
local secure_obj = object:new({
    proto = 'https',
    host = 'influxdb.example.com',
    port = 8086,
    db = 'production',
    ssl = true,
    auth = 'admin:secret'
})
```

### set_measurement

设置测量名称

#### 函数原型

```lua
function obj:set_measurement(measurement)
end
```

#### 参数说明

* measurement
  测量名称字符串

#### 示例

```lua
obj:set_measurement('temperature')
obj:set_measurement('http_requests')
```

### add_tag

添加标签

#### 函数原型

```lua
function obj:add_tag(key, value)
end
```

#### 参数说明

* key
  标签键
* value
  标签值

#### 注意事项

> 标签会被索引，适合用于筛选和分组。避免使用高基数标签。

#### 示例

```lua
obj:set_measurement('temperature')

-- 添加多个标签
obj:add_tag('location', 'room1')
obj:add_tag('sensor', 'S001')
obj:add_tag('type', 'DHT22')

-- 添加字段后写入
obj:add_field('value', 25.5)
obj:write()
```

### add_field

添加字段

#### 函数原型

```lua
function obj:add_field(key, value)
end
```

#### 参数说明

* key
  字段键
* value
  字段值（数字、字符串、布尔值）

#### 注意事项

> 至少需要一个字段才能写入数据。

#### 示例

```lua
-- 添加数值字段
obj:add_field('value', 25.5)
obj:add_field('count', 100)

-- 添加字符串字段
obj:add_field('status', 'OK')
obj:add_field('message', 'Normal operation')

-- 添加布尔字段
obj:add_field('active', true)
obj:add_field('online', false)
```

### stamp

设置或获取时间戳

#### 函数原型

```lua
function obj:stamp(time)
end
```

#### 参数说明

* time
  可选，指定时间戳（数字）

#### 返回值

返回时间戳字符串

#### 说明

- 不提供参数时：根据precision自动生成时间戳
- 提供参数时：设置指定的时间戳

#### 示例

```lua
-- 自动生成时间戳
obj:stamp()

-- 设置自定义时间戳
obj:stamp(1609459200000)  -- 毫秒时间戳
obj:stamp(1609459200)     -- 秒时间戳
```

### timestamp

获取时间戳

#### 函数原型

```lua
function obj:timestamp()
end
```

#### 返回值

返回时间戳字符串。如果时间戳不存在则自动生成。

#### 示例

```lua
local ts = obj:timestamp()
print('Timestamp:', ts)
```

### clear

清空对象数据

#### 函数原型

```lua
function obj:clear()
end
```

#### 返回值

成功返回true

#### 说明

清空后可以重新构建数据，重用对象。

#### 示例

```lua
-- 第一次使用
obj:set_measurement('temperature')
obj:add_tag('location', 'room1')
obj:add_field('value', 25.5)
obj:write()

-- 清空后重用
obj:clear()

-- 第二次使用
obj:set_measurement('humidity')
obj:add_tag('location', 'room1')
obj:add_field('value', 65.2)
obj:write()
```

### write

立即写入当前数据

#### 函数原型

```lua
function obj:write()
end
```

#### 返回值

成功返回true，失败返回false和错误信息

#### 说明

此方法会：
1. 验证数据完整性（必须有measurement和fields）
2. 生成时间戳（如果未设置）
3. 构建行协议
4. 执行写入
5. 清空数据（调用clear()）

#### 示例

```lua
obj:set_measurement('temperature')
obj:add_tag('sensor', 'S001')
obj:add_field('value', 25.5)

local ok, err = obj:write()
if not ok then
    print('Write failed:', err)
end
```

### buffer_ready

检查是否准备好缓冲

#### 函数原型

```lua
function obj:buffer_ready()
end
```

#### 返回值

返回boolean和错误信息：
- true: 准备好
- false, error_msg: 未准备好，包含错误原因

#### 检查条件

- 必须设置了measurement
- 必须至少有一个field

#### 示例

```lua
local ready, err = obj:buffer_ready()
if not ready then
    print('Not ready:', err)
end
```

### buffer

将当前数据添加到缓冲区

#### 函数原型

```lua
function obj:buffer()
end
```

#### 返回值

成功返回true，失败返回false和错误信息

#### 说明

此方法会：
1. 验证数据完整性
2. 构建行协议
3. 添加到内部缓冲区
4. 清空当前数据（调用clear()）

#### 示例

```lua
-- 添加多个数据点到缓冲区
for i = 1, 10 do
    obj:set_measurement('metric')
    obj:add_tag('id', tostring(i))
    obj:add_field('value', math.random())
    obj:buffer()
end

-- 稍后刷新缓冲区
obj:flush()
```

### flush_ready

检查是否准备好刷新

#### 函数原型

```lua
function obj:flush_ready()
end
```

#### 返回值

返回boolean和错误信息

#### 检查条件

- 不能有未缓冲的measurement
- 不能有未缓冲的fields
- 必须有缓冲的数据

#### 示例

```lua
local ready, err = obj:flush_ready()
if not ready then
    print('Cannot flush:', err)
end
```

### flush

刷新缓冲区，写入所有缓冲的数据

#### 函数原型

```lua
function obj:flush()
end
```

#### 返回值

成功返回true，失败返回false和错误信息

#### 说明

此方法会：
1. 验证缓冲区有数据
2. 合并所有缓冲的数据点
3. 执行批量写入
4. 清空缓冲区

#### 示例

```lua
-- 收集多个数据点
for i = 1, 100 do
    obj:set_measurement('temperature')
    obj:add_field('value', math.random() * 30)
    obj:buffer()
end

-- 批量写入
local ok, err = obj:flush()
if not ok then
    print('Flush failed:', err)
end
```

## 完整示例

### 基本使用

```lua
local object = require 'db.influxdb.object'

local obj = object:new({
    proto = 'http',
    host = 'localhost',
    port = 8086,
    db = 'sensors',
    precision = 'ms'
})

-- 写入温度数据
obj:set_measurement('temperature')
obj:add_tag('location', 'server_room')
obj:add_tag('sensor', 'DHT22_01')
obj:add_field('temperature', 25.5)
obj:add_field('humidity', 65.2)

obj:write()
print('Temperature data written')
```

### 批量写入

```lua
local obj = object:new({
    proto = 'http',
    host = 'localhost',
    port = 8086,
    db = 'metrics'
})

-- 收集多个指标
local metrics = {
    {name='cpu_usage', tags={cpu='cpu0'}, value=45.2},
    {name='cpu_usage', tags={cpu='cpu1'}, value=32.8},
    {name='memory_usage', tags={}, value=68.5},
    {name='disk_usage', tags={mount='/'}, value=82.3},
}

-- 批量缓冲
for _, m in ipairs(metrics) do
    obj:set_measurement(m.name)
    for k, v in pairs(m.tags) do
        obj:add_tag(k, v)
    end
    obj:add_field('value', m.value)
    obj:buffer()
end

-- 一次性写入
obj:flush()
```

### 对象重用

```lua
local obj = object:new({
    proto = 'http',
    host = 'localhost',
    port = 8086,
    db = 'production'
})

-- 设置公共标签
local function write_metric(name, value, extra_tags)
    obj:set_measurement(name)
    obj:add_tag('host', 'server01')
    obj:add_tag('region', 'us-west')

    -- 添加额外标签
    if extra_tags then
        for k, v in pairs(extra_tags) do
            obj:add_tag(k, v)
        end
    end

    obj:add_field('value', value)
    obj:write()
end

-- 写入多个指标
write_metric('cpu_usage', 45.2, {cpu='cpu0'})
write_metric('cpu_usage', 32.8, {cpu='cpu1'})
write_metric('memory_usage', 68.5)
write_metric('disk_usage', 82.3, {mount='/'})
```

### 数据收集器

```lua
local object = require 'db.influxdb.object'

local collector = object:new({
    proto = 'http',
    host = 'localhost',
    port = 8086,
    db = 'telemetry'
})

-- 收集传感器数据
local function collect_sensor_data(sensor_id, readings)
    for _, reading in ipairs(readings) do
        collector:set_measurement('sensor_reading')
        collector:add_tag('sensor_id', sensor_id)
        collector:add_tag('type', reading.type)
        collector:add_field('value', reading.value)
        collector:buffer()
    end
end

-- 模拟传感器数据
local sensor_data = {
    {
        sensor_id = 'S001',
        readings = {
            {type='temperature', value=25.5},
            {type='humidity', value=65.2},
            {type='pressure', value=1013.25}
        }
    },
    {
        sensor_id = 'S002',
        readings = {
            {type='temperature', value=24.8},
            {type='humidity', value=63.8}
        }
    }
}

-- 收集所有数据
for _, sensor in ipairs(sensor_data) do
    collect_sensor_data(sensor.sensor_id, sensor.readings)
end

-- 批量写入
collector:flush()
```

### 自定义时间戳

```lua
local obj = object:new({
    proto = 'http',
    host = 'localhost',
    port = 8086,
    db = 'historical',
    precision = 'ms'
})

-- 写入历史数据
local historical_data = {
    {value=25.5, ts=1609459200000},
    {value=26.1, ts=1609459260000},
    {value=24.8, ts=1609459320000},
}

for _, data in ipairs(historical_data) do
    obj:set_measurement('temperature')
    obj:add_tag('sensor', 'S001')
    obj:add_field('value', data.value)

    -- 设置历史时间戳
    obj:stamp(data.ts)

    obj:write()
end
```

### UDP写入

```lua
local object = require 'db.influxdb.object'

-- 创建UDP对象
local udp_obj = object:new({
    proto = 'udp',
    host = 'localhost',
    port = 8089,
    db = 'high_frequency',
    precision = 'ms'
})

-- 高频数据写入
for i = 1, 1000 do
    udp_obj:set_measurement('vibration')
    udp_obj:add_tag('axis', 'x')
    udp_obj:add_field('value', math.random())

    -- UDP可以立即写入
    udp_obj:write()
end
```

### 错误处理

```lua
local obj = object:new({
    proto = 'http',
    host = 'localhost',
    port = 8086,
    db = 'test'
})

-- 安全写入函数
local function safe_write(obj)
    local ok, err = obj:write()
    if not ok then
        print('Write failed:', err)

        -- 检查错误类型
        if err:match('no measurement') then
            print('Error: Measurement not set')
        elseif err:match('no fields') then
            print('Error: No fields to write')
        else
            print('Unknown error:', err)
        end

        return false
    end

    return true
end

-- 使用
obj:set_measurement('test')
obj:add_field('value', 100)
safe_write(obj)
```

## 高级用法

### 数据验证

```lua
-- 检查对象状态
local function check_object(obj)
    -- 检查是否准备好写入
    local ready, err = obj:buffer_ready()
    if not ready then
        print('Not ready to write:', err)
        return false
    end

    -- 检查时间戳
    local ts = obj:timestamp()
    print('Timestamp:', ts)

    return true
end

obj:set_measurement('test')
obj:add_field('value', 100)

if check_object(obj) then
    obj:write()
end
```

### 批量管理

```lua
local function batch_write(obj, data_list, batch_size)
    batch_size = batch_size or 100

    for i, data in ipairs(data_list) do
        obj:set_measurement(data.measurement)

        -- 添加标签
        if data.tags then
            for k, v in pairs(data.tags) do
                obj:add_tag(k, v)
            end
        end

        -- 添加字段
        if data.fields then
            for k, v in pairs(data.fields) do
                obj:add_field(k, v)
            end
        end

        obj:buffer()

        -- 达到批量大小时刷新
        if i % batch_size == 0 then
            print('Flushing batch', i / batch_size)
            obj:flush()
        end
    end

    -- 刷新剩余数据
    obj:flush()
end

-- 使用
local data = {}
for i = 1, 250 do
    table.insert(data, {
        measurement = 'metric',
        tags = {id = tostring(i)},
        fields = {value = math.random()}
    })
end

batch_write(obj, data, 100)
```

## 性能优化

### 批量 vs 立即写入

```lua
-- 批量写入（推荐用于大量数据）
for i = 1, 1000 do
    obj:set_measurement('metric')
    obj:add_field('value', math.random())
    obj:buffer()
end
obj:flush()

-- 立即写入（推荐用于少量关键数据）
obj:set_measurement('alert')
obj:add_field('message', 'Critical error')
obj:write()
```

### 对象池

```lua
-- 创建对象池重用对象
local object_pool = {}

local function get_object()
    local obj = table.remove(object_pool)
    if not obj then
        obj = object:new({
            proto = 'http',
            host = 'localhost',
            port = 8086,
            db = 'production'
        })
    end
    return obj
end

local function release_object(obj)
    obj:clear()
    table.insert(object_pool, obj)
end

-- 使用
local obj = get_object()
obj:set_measurement('metric')
obj:add_field('value', 100)
obj:write()
release_object(obj)
```

## 最佳实践

1. **对象重用**：使用clear()重用对象而非创建新对象
2. **批量写入**：大量数据使用buffer()和flush()
3. **错误处理**：检查返回值并处理错误
4. **标签设计**：避免高基数标签
5. **时间戳**：需要精确时间时手动设置
6. **协议选择**：高频数据使用UDP，关键数据使用HTTP

## 注意事项

1. **状态管理**：write()和buffer()会清空数据
2. **批量限制**：大量缓冲会占用内存
3. **时间精度**：确保时间戳与precision匹配
4. **字段类型**：InfluxDB会推断字段类型
5. **标签索引**：标签会建立索引，字段不会

## 与其他模块配合

```lua
-- 与buffer模块配合
local buffer = require 'db.influxdb.buffer'
local object = require 'db.influxdb.object'

-- 使用object构建数据
local obj = object:new({...})
obj:set_measurement('metric')
obj:add_field('value', 100)

-- 使用buffer的接口写入
-- 需要手动构建数据格式
```
