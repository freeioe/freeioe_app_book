---

# Prometheus指标模块 (db.prometheus.metric)

本模块提供Prometheus指标（Metric）数据结构，用于组织和编码单个指标的数据点。

## 使用方法

```lua
local pmetric = require 'db.prometheus.metric'

-- 创建指标
local metric = pmetric:new('http_requests_total', {
    method = 'GET',
    status = '200'
}, 'counter', 'Total HTTP requests')

-- 添加数据点
metric:push_value(1, timestamp)

-- 编码为Prometheus文本格式
local lines = metric:encode(true)
```

## 接口说明

### new

创建指标对象

#### 函数原型

```lua
function pmetric:new(name, labels, typ, help)
end
```

#### 参数说明

* name
  指标名称字符串，必须符合Prometheus命名规范
* labels
  可选，标签表（键值对）
* typ
  可选，指标类型（gauge, counter, histogram, summary）
* help
  可选，指标描述信息

#### 指标命名规范

- 必须匹配正则：`[a-zA-Z_:][a-zA-Z0-9_:]*`
- 推荐使用下划线分隔
- 使用有意义的名称

#### 返回值

返回指标对象

### metric_name

获取指标名称

#### 函数原型

```lua
function metric:metric_name()
end
```

#### 返回值

返回指标名称字符串

### labels

获取或设置标签

#### 获取标签

```lua
function metric:labels()
end
```

返回当前标签表

#### 设置标签

```lua
function metric:set_label(name, value)
end
```

参数：
- name：标签名
- value：标签值

### push_value

添加数据点到指标

#### 函数原型

```lua
function metric:push_value(value, timestamp)
end
```

#### 参数说明

* value
  数值（会被转换为number类型）
* timestamp
  可选，时间戳（默认使用skynet.time()）

#### 值转换

```lua
-- 字符串会转换为数字
metric:push_value("100")  --> 100.0

-- nil会转换为0
metric:push_value(nil)    --> 0.0

-- 布尔值会转换
metric:push_value(true)   --> 1.0
metric:push_value(false)  --> 0.0
```

### values

获取所有数据点

#### 函数原型

```lua
function metric:values()
end
```

#### 返回值

返回数据点数组，每个元素格式为：
```lua
{
    value = number,
    timestamp = number
}
```

### clean

清空所有数据点

#### 函数原型

```lua
function metric:clean()
end
```

### encode

编码指标为Prometheus文本格式

#### 函数原型

```lua
function metric:encode(auto_clean)
end
```

#### 参数说明

* auto_clean
  可选，编码后是否自动清空数据（默认false）

#### 返回值

返回字符串数组，格式为Prometheus文本协议

#### 输出格式

```lua
{
    '# TYPE metric_name counter',
    '# HELP metric_name Metric description',
    'metric_name{label1="value1",label2="value2"} value timestamp',
    'metric_name{label1="value1",label2="value2"} value timestamp',
    ...
}
```

#### 示例

```lua
local metric = pmetric:new('temperature', {
    sensor_id = 'S001',
    location = 'room1'
}, 'gauge', 'Temperature in Celsius')

metric:push_value(25.5, 1609459200)
metric:push_value(26.1, 1609459260)

local lines = metric:encode()

-- 输出：
-- {
--     '# TYPE temperature gauge',
--     '# HELP temperature Temperature in Celsius',
--     'temperature{sensor_id="S001",location="room1"} 25.5 1609459200000',
--     'temperature{sensor_id="S001",location="room1"} 26.1 1609459260000'
-- }
```

## 完整示例

### Counter类型指标

```lua
local pmetric = require 'db.prometheus.metric'

-- 请求计数器
local counter = pmetric:new('http_requests_total', {
    method = 'GET',
    status = '200'
}, 'counter', 'Total HTTP requests')

-- 增加计数
counter:push_value(1, timestamp1)
counter:push_value(1, timestamp2)  -- 总数增加

-- 编码
local lines = counter:encode(true)
print(table.concat(lines, '\n'))

-- 输出：
-- # TYPE http_requests_total counter
-- # HELP http_requests_total Total HTTP requests
-- http_requests_total{method="GET",status="200"} 2 1609459200000
```

### Gauge类型指标

```lua
-- 温度仪表
local gauge = pmetric:new('sensor_temperature_celsius', {
    sensor_id = 'S001',
    location = 'server_room'
}, 'gauge', 'Temperature sensor reading')

-- 设置当前值
gauge:push_value(25.5, timestamp1)
gauge:push_value(26.1, timestamp2)  -- 当前温度

-- 编码
local lines = gauge:encode(true)
```

### Histogram类型指标（简化）

```lua
-- 延迟直方图
local function create_latency_histogram()
    -- 创建多个bucket指标
    local buckets = {0.005, 0.01, 0.025, 0.05, 0.1, 0.25, 0.5, 1, 2.5, 5, 10}

    local metrics = {}
    for _, le in ipairs(buckets) do
        local m = pmetric:new('http_request_duration_seconds_bucket', {
            le = tostring(le),
            endpoint = '/api/users'
        }, 'counter')
        table.insert(metrics, m)
    end

    -- +Inf bucket
    local inf = pmetric:new('http_request_duration_seconds_bucket', {
        le = '+Inf',
        endpoint = '/api/users'
    }, 'counter')
    table.insert(metrics, inf)

    return metrics
end

-- 观察延迟值
local function observe_latency(metrics, duration)
    local count = 0

    for _, m in ipairs(metrics) do
        local le = m:labels().le
        if le == '+Inf' or tonumber(le) >= duration then
            m:push_value(1, skynet.time())
            count = count + 1
        end
    end
end
```

### 带标签的指标

```lua
-- 创建带多个标签的指标
local metric = pmetric:new('api_request_duration_seconds', {
    method = 'POST',
    endpoint = '/api/users',
    status = '201',
    version = 'v1'
}, 'histogram', 'API request duration')

metric:push_value(0.123, timestamp)

-- 动态修改标签
metric:set_label('version', 'v2')
metric:push_value(0.089, timestamp)
```

### 批量数据点

```lua
-- 收集多个数据点
local metric = pmetric:new('cpu_usage_percent', {
    cpu = 'cpu0'
}, 'gauge')

for i = 1, 60 do
    local usage = math.random(20, 80)
    local ts = timestamp - (60 - i)
    metric:push_value(usage, ts)
end

-- 编码并清理
local lines = metric:encode(true)
print('Encoded', #lines - 2, 'data points')  -- 减去TYPE和HELP行
```

## 标签值转义

特殊字符会被自动转义：

```lua
local metric = pmetric:new('test', {
    key1 = 'value with "quotes"',
    key2 = 'value\nwith\nnewlines',
    key3 = 'value\\with\\backslash'
})

-- 编码后：
-- test{key1="value with \\"quotes\\"",key2="value\\nwith\\nnewlines",key3="value\\\\with\\\\backslash"} 0 1609459200000
```

## 指标类型说明

### Counter（计数器）

- **特性**：只能递增，不会减少
- **用途**：请求总数、错误总数等
- **命名**：通常以`_total`结尾
- **示例**：
  - `http_requests_total`
  - `errors_total`
  - `bytes_sent_total`

### Gauge（仪表）

- **特性**：可增可减
- **用途**：当前值、状态等
- **命名**：描述当前状态
- **示例**：
  - `temperature_celsius`
  - `memory_usage_bytes`
  - `active_connections`

### Histogram（直方图）

- **特性**：统计分布
- **用途**：延迟分布、请求大小分布等
- **命名**：通常包含`_bucket`, `_sum`, `_count`
- **示例**：
  - `http_request_duration_seconds_bucket`
  - `response_size_bytes_bucket`

### Summary（摘要）

- **特性**：客户端计算的统计信息
- **用途**：预计算的百分位数
- **命名**：通常包含`_sum`, `_count`, 以及分位数
- **示例**：
  - `rpc_duration_seconds`
  - `latency_seconds`

## 与数据容器配合

```lua
local pmetric = require 'db.prometheus.metric'
local pdata = require 'db.prometheus.data'

local data = pdata:new()

-- 添加多个指标
local metric1 = pmetric:new('metric1', nil, 'gauge')
metric1:push_value(100, ts)
data:add_metric(metric1)

local metric2 = pmetric:new('metric2', nil, 'counter')
metric2:push_value(1, ts)
data:add_metric(metric2)

-- 批量插入
db:insert(data, true)
```

## 最佳实践

1. **命名规范**：
   - 使用有意义的名称
   - 包含单位（如`_seconds`, `_bytes`）
   - Counter以`_total`结尾

2. **标签设计**：
   - 避免高基数标签（如user_id）
   - 使用一致的标签键名
   - 标签值数量要合理

3. **指标类型**：
   - 正确选择指标类型
   - Counter只增不减
   - Gauge可增可减

4. **HELP信息**：
   - 提供清晰的描述
   - 说明单位
   - 解释指标含义

5. **性能优化**：
   - 批量收集数据点
   - 定期清理已编码的数据
   - 避免过多的标签组合
