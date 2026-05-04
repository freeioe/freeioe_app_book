---

# SiriDB数据容器模块 (db.siridb.data)

本模块提供SiriDB数据容器，用于组织和编码多个时序（Series）的数据。

## 使用方法

```lua
local sdata = require 'db.siridb.data'
local sseries = require 'db.siridb.series'

-- 创建数据容器
local data = sdata:new()

-- 创建时序
local series1 = sseries:new('temperature', 'float')
series1:push_value(25.5, timestamp)

local series2 = sseries:new('humidity', 'float')
series2:push_value(65.2, timestamp)

-- 添加到容器
data:add_series('temperature', series1)
data:add_series('humidity', series2)

-- 编码并插入数据库
db:insert(data, true)
```

## 接口说明

### new

创建数据容器对象

#### 函数原型

```lua
function sdata:new()
end
```

#### 返回值

返回数据容器对象

### add_series

添加时序到容器

#### 函数原型

```lua
function data:add_series(name, series)
end
```

#### 参数说明

* name
  时序名称字符串，用作容器的键
* series
  时序对象

#### 注意事项

> 时序名称必须在容器中唯一，重复添加同名时序会失败。

#### 示例

```lua
local data = sdata:new()

local temp = sseries:new('temperature', 'float')
data:add_series('temperature', temp)

-- 尝试添加同名时序会失败
local temp2 = sseries:new('temperature', 'float')
data:add_series('temperature', temp2)  -- assert失败
```

### list

获取容器中的所有时序

#### 函数原型

```lua
function data:list()
end
```

#### 返回值

返回时序表，键为时序名称，值为时序对象

### encode

编码容器中所有时序的数据

#### 函数原型

```lua
function data:encode(time_precision, auto_clean)
end
```

#### 参数说明

* time_precision
  时间精度（'s', 'ms', 'us', 'ns'）
* auto_clean
  可选，编码后是否自动清空各时序的数据（默认false）

#### 返回值

返回编码后的数据表，格式为：
```lua
{
    series_name1 = {{ts1, val1}, {ts2, val2}, ...},
    series_name2 = {{ts1, val1}, {ts2, val2}, ...},
    ...
}
```

## 完整示例

### 多传感器数据采集

```lua
local skynet = require 'skynet'
local sdata = require 'db.siridb.data'
local sseries = require 'db.siridb.series'

-- 创建数据容器
local data = sdata:new()

-- 创建多个时序
local sensors = {
    {name='temp_01', type='float', value=25.5},
    {name='temp_02', type='float', value=24.8},
    {name='humidity_01', type='float', value=65.2},
    {name='pressure_01', type='float', value=1013.25},
}

-- 添加所有时序到容器
for _, sensor in ipairs(sensors) do
    local series = sseries:new(sensor.name, sensor.type)
    series:push_value(sensor.value, skynet.time())
    data:add_series(sensor.name, series)
end

-- 一次性插入数据库
db:insert(data, true)
```

### 批量数据收集

```lua
local sdata = require 'db.siridb.data'

-- 批量收集函数
local function collect_batch(readings)
    local data = sdata:new()

    for sensor_id, value in pairs(readings) do
        local series_name = string.format('sensor_%s', sensor_id)
        local series = sseries:new(series_name, 'float')
        series:push_value(value, skynet.time())
        data:add_series(series_name, series)
    end

    return data
end

-- 收集传感器数据
local readings = {
    ['001'] = 25.3,
    ['002'] = 24.9,
    ['003'] = 26.1,
    ['004'] = 25.7,
}

local batch = collect_batch(readings)
db:insert(batch, true)
```

### 不同类型的数据

```lua
local sdata = require 'db.siridb.data'
local sseries = require 'db.siridb.series'

local data = sdata:new()

-- 添加不同类型的时序
local counter = sseries:new('request_count', 'int')
counter:push_value(100, timestamp)
data:add_series('counter', counter)

local temperature = sseries:new('temperature', 'float')
temperature:push_value(25.5, timestamp)
data:add_series('temperature', temperature)

local status = sseries:new('system_status', 'string')
status:push_value('RUNNING', timestamp)
data:add_series('status', status)

-- 编码所有数据
local encoded = data:encode('ms', true)
```

## 高级用法

### 动态时序管理

```lua
local sdata = require 'db.siridb.data'

local container = sdata:new()

-- 动态添加时序
local function add_reading(name, value, timestamp)
    local list = container:list()

    local series = list[name]
    if not series then
        -- 创建新时序
        series = sseries:new(name, 'float')
        container:add_series(name, series)
    end

    -- 添加数据点
    series:push_value(value, timestamp)
end

-- 使用
add_reading('temp_01', 25.5, ts1)
add_reading('temp_02', 24.8, ts2)
add_reading('temp_01', 26.1, ts3)  -- 添加到已存在的时序

-- 插入数据库
db:insert(container, true)
```

### 条件编码

```lua
local data = sdata:new()

-- 添加多个时序
-- ... 添加时序的代码 ...

-- 仅当有足够数据时才编码
local function should_encode(data)
    local list = data:list()
    local total_points = 0

    for name, series in pairs(list) do
        total_points = total_points + #series:values()
    end

    return total_points >= 100  -- 至少100个数据点
end

if should_encode(data) then
    db:insert(data, true)
end
```

### 数据预览

```lua
local data = sdata:new()
-- ... 添加时序 ...

-- 预览数据而不编码
local list = data:list()
for name, series in pairs(list) do
    local values = series:values()
    print(string.format('Series: %s, Points: %d',
        name, #values))

    -- 打印前3个数据点
    for i, v in ipairs(values) do
        if i > 3 then break end
        print(string.format('  [%d] %s: %f',
            v[1], os.date('%Y-%m-%d %H:%M:%S', v[1]), v[2]))
    end
end
```

## 与database模块配合

```lua
local sdata = require 'db.siridb.data'
local sseries = require 'db.siridb.series'
local siridb_db = require 'db.siridb.database'

-- 创建数据库连接
local db = siridb_db:new({
    host = '127.0.0.1',
    port = 9020
}, 'production')

-- 收集数据函数
local function collect_and_insert()
    local data = sdata:new()

    -- 模拟从多个传感器收集数据
    for i = 1, 10 do
        local series_name = string.format('sensor_%d', i)
        local series = sseries:new(series_name, 'float')

        -- 每个时序添加多个数据点
        for j = 1, 10 do
            local value = 20 + math.random() * 10
            local ts = skynet.time() - (10 - j) * 60  -- 过去10分钟
            series:push_value(value, ts)
        end

        data:add_series(series_name, series)
    end

    -- 一次性插入所有数据
    local ok, err = db:insert(data, true)
    if not ok then
        log:error('Insert failed:', err)
    end

    return ok
end
```

## 性能优化

### 批量大小控制

```lua
local data = sdata:new()

-- 检查批量大小
local function get_total_points(data)
    local total = 0
    local list = data:list()

    for _, series in pairs(list) do
        total = total + #series:values()
    end

    return total
end

-- 当达到一定大小时插入
if get_total_points(data) >= 1000 then
    db:insert(data, true)
end
```

### 内存管理

```lua
local data = sdata:new()

-- 使用auto_clean自动清理
db:insert(data, true)  -- 插入后自动清理所有时序

-- 或者手动清理
local list = data:list()
for _, series in pairs(list) do
    series:clean()
end
```

## 错误处理

```lua
local data = sdata:new()

-- 添加时序时检查重复
local function safe_add_series(data, name, series)
    local list = data:list()

    if list[name] then
        log.warning('Series already exists:', name)
        return false
    end

    data:add_series(name, series)
    return true
end

-- 使用
local series = sseries:new('test', 'float')
if safe_add_series(data, 'test', series) then
    print('Series added successfully')
end
```

## 最佳实践

1. **批量操作**：使用容器批量插入多个时序
2. **唯一命名**：确保容器中时序名称唯一
3. **自动清理**：插入后使用auto_clean清理内存
4. **大小控制**：监控批量大小，避免单次插入过多数据
5. **错误处理**：检查返回值并处理错误
