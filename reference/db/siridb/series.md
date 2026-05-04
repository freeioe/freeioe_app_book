---

# SiriDB时序数据模块 (db.siridb.series)

本模块提供SiriDB时序（Series）数据结构，用于组织和编码单个时序的数据点。

## 使用方法

```lua
local sseries = require 'db.siridb.series'

-- 创建时序对象
local series = sseries:new('temperature', 'float')

-- 添加数据点
series:push_value(25.5, skynet.time())
series:push_value(26.1, skynet.time())
series:push_value(24.8, skynet.time())

-- 获取所有数据点
local values = series:values()

-- 编码为数据库格式
local encoded = series:encode('ms')

-- 清空数据
series:clean()
```

## 接口说明

### new

创建时序对象

#### 函数原型

```lua
function sseries:new(name, value_type)
end
```

#### 参数说明

* name
  时序名称字符串
* value_type
  可选，值类型（默认'float'）

| 类型 | 说明 | 示例 |
| :--- | :--- | :--- |
| 'int' | 整数 | 42 |
| 'float' | 浮点数 | 3.14 |
| 'string' | 字符串 | "error" |

#### 返回值

返回时序对象

### series_name

获取时序名称

#### 函数原型

```lua
function series:series_name()
end
```

#### 返回值

返回时序名称字符串

### value_type

获取值类型

#### 函数原型

```lua
function series:value_type()
end
```

#### 返回值

返回值类型字符串（'int', 'float', 'string'）

### push_value

添加数据点到时序

#### 函数原型

```lua
function series:push_value(value, timestamp)
end
```

#### 参数说明

* value
  数据值，类型取决于value_type
* timestamp
  可选，时间戳（默认使用skynet.time()）

#### 值类型转换

- **int**：自动取整
- **float**：转换为浮点数
- **string**：转换为字符串，转换失败则为"ERROR_STRING"

#### 示例

```lua
-- 整数时序
local int_series = sseries:new('counter', 'int')
int_series:push_value(100)           -- 100
int_series:push_value(3.14)          -- 3
int_series:push_value("42")          -- 42

-- 浮点数时序
local float_series = sseries:new('temperature', 'float')
float_series:push_value(25.5)        -- 25.5
float_series:push_value(100)         -- 100.0
float_series:push_value("26.8")      -- 26.8

-- 字符串时序
local string_series = sseries:new('status', 'string')
string_series:push_value("OK")       -- "OK"
string_series:push_value(100)        -- "100"
string_series:push_value(nil)        -- "nil"
```

### values

获取所有数据点

#### 函数原型

```lua
function series:values()
end
```

#### 返回值

返回数据点数组，每个元素格式为 `{timestamp, value}`

### clean

清空所有数据点

#### 函数原型

```lua
function series:clean()
end
```

### encode

编码时序数据为数据库格式

#### 函数原型

```lua
function series:encode(time_precision, auto_clean)
end
```

#### 参数说明

* time_precision
  时间精度（'s', 'ms', 'us', 'ns'）
* auto_clean
  可选，编码后是否自动清空数据（默认false）

#### 返回值

返回编码后的数据表，格式为：
```lua
{
    {timestamp1, value1},
    {timestamp2, value2},
    ...
}
```

时间戳将根据time_precision转换为相应精度。

## 完整示例

### 温度传感器数据

```lua
local skynet = require 'skynet'
local sseries = require 'db.siridb.series'

-- 创建温度时序
local temp_series = sseries:new('sensor_temp_01', 'float')

-- 模拟传感器读数
local function read_sensor()
    -- 返回模拟温度值（20-30度之间）
    return 20 + math.random() * 10
end

-- 每秒采集一次数据
for i = 1, 60 do
    local temp = read_sensor()
    temp_series:push_value(temp, skynet.time())
    skynet.sleep(100)  -- 等待1秒
end

-- 获取所有数据
local all_values = temp_series:values()
print('Total readings:', #all_values)

-- 计算平均值
local sum = 0
for _, v in ipairs(all_values) do
    sum = sum + v[2]
end
print('Average temperature:', sum / #all_values)

-- 编码并插入数据库
local encoded = temp_series:encode('ms', true)  -- 编码后清空
print('Encoded data points:', #encoded)
```

### 计数器数据

```lua
local skynet = require 'skynet'
local sseries = require 'db.siridb.series'

-- 创建计数器时序
local counter_series = sseries:new('request_count', 'int')

-- 记录请求计数
local function record_requests(count)
    counter_series:push_value(count, skynet.time())
end

-- 模拟请求
record_requests(10)
record_requests(15)
record_requests(8)

-- 编码为秒级精度
local encoded = counter_series:encode('s')

-- 查看编码后的数据
for i, v in ipairs(encoded) do
    print(string.format('%d: timestamp=%d, value=%d',
        i, v[1], v[2]))
end
```

### 状态日志

```lua
local skynet = require 'skynet'
local sseries = require 'db.siridb.series'

-- 创建状态时序
local status_series = sseries:new('system_status', 'string')

-- 记录状态变化
status_series:push_value('STARTING', skynet.time())
skynet.sleep(100)
status_series:push_value('RUNNING', skynet.time())
skynet.sleep(100)
status_series:push_value('WARNING', skynet.time())
skynet.sleep(100)
status_series:push_value('ERROR', skynet.time())

-- 获取状态历史
local history = status_series:values()
for _, event in ipairs(history) do
    print(string.format('[%s] Status: %s',
        os.date('%Y-%m-%d %H:%M:%S', event[1]),
        event[2]))
end
```

## 数据类型处理

### 类型转换规则

| 输入类型 | int | float | string |
| :--- | :--- | :--- | :--- |
| number | math.floor() | tonumber() | tostring() |
| string | tonumber() | tonumber() | 原值 |
| boolean | - | - | tostring() |
| nil | 0 | 0.0 | "nil" |
| table | - | - | "table" |

### 特殊值处理

```lua
-- NaN和Inf处理（float类型）
series:push_value(1/0)       --> Inf
series:push_value(0/0)       --> NaN
series:push_value(math.huge) --> Inf

-- 空字符串处理（string类型）
series:push_value("")        --> ""
series:push_value(nil)       --> "nil"
```

## 性能优化

### 批量添加

```lua
-- 低效：逐个添加
for i = 1, 1000 do
    series:push_value(value, timestamp)
end

-- 高效：预分配数组（内部实现）
local values = {}
for i = 1, 1000 do
    table.insert(values, {timestamp, value})
end
-- 直接操作内部数据（不推荐，仅示例）
```

### 内存管理

```lua
-- 定期清理已编码的数据
local series = sseries:new('metric', 'float')

-- 添加数据
series:push_value(1.0, ts1)
series:push_value(2.0, ts2)

-- 编码并自动清理
db:insert_series(series, true)  -- auto_clean=true

-- 或者手动清理
series:clean()
```

## 与数据容器配合使用

```lua
local sseries = require 'db.siridb.series'
local sdata = require 'db.siridb.data'

-- 创建数据容器
local container = sdata:new()

-- 创建多个时序
local temp1 = sseries:new('temp_sensor_1', 'float')
local temp2 = sseries:new('temp_sensor_2', 'float')
local humidity = sseries:new('humidity_sensor_1', 'float')

-- 添加数据
temp1:push_value(25.5, ts)
temp2:push_value(24.8, ts)
humidity:push_value(65.2, ts)

-- 添加到容器
container:add_series('temp_sensor_1', temp1)
container:add_series('temp_sensor_2', temp2)
container:add_series('humidity_sensor_1', humidity)

-- 批量插入
db:insert(container, true)
```

## 错误处理

```lua
local series = sseries:new('test', 'float')

-- push_value不会失败，值会被转换
series:push_value("invalid")  --> 转换为0.0
series:push_value(nil)        --> 转换为0.0

-- 检查转换后的值
local values = series:values()
for _, v in ipairs(values) do
    print(v[2])  -- 检查实际值
end
```

## 最佳实践

1. **命名规范**：使用描述性的时序名称
   - 好：`'temp_sensor_01'`
   - 差：`'t1'`

2. **类型选择**：根据数据特性选择合适的值类型
   - 计数器：使用'int'
   - 测量值：使用'float'
   - 状态：使用'string'

3. **时间戳**：使用一致的时间戳来源
   - 推荐：`skynet.time()`
   - 避免：混合使用不同时间源

4. **批量操作**：积累多个数据点后再插入
   - 不要：每个数据点立即插入
   - 推荐：批量插入提高效率
