---

# Prometheus数据库模块 (db.prometheus.database)

本模块提供Prometheus数据库的客户端接口，支持指标写入、查询和PromQL操作。

## 使用方法

```lua
local prom_db = require 'db.prometheus.database'

-- 创建数据库连接
local db = prom_db:new({
    host = '127.0.0.1',
    port = 9090,
    job = 'myapp',
    instance = 'localhost:8080'
})

-- 插入指标
local ok, err = db:insert(data)

-- 查询指标
local result, err = db:query('up')

-- 范围查询
local result, err = db:query_range('rate(http_requests_total[5m]', start, end, '1m')
```

## 接口说明

### new

创建Prometheus数据库对象

#### 函数原型

```lua
function prom_db:new(options)
end
```

#### 参数说明

* options
  连接选项表

| 选项 | 类型 | 默认值 | 说明 |
| :--- | :--- | :--- | :--- |
| host | string | '127.0.0.1' | Prometheus服务器地址 |
| port | number | 9090 | Prometheus端口 |
| ssl | boolean | false | 是否使用SSL/TLS |
| username | string | nil | 认证用户名 |
| password | string | nil | 认证密码 |
| timeout | number | nil | 请求超时（毫秒） |
| url | string | '/metric' | 指标推送URL路径 |
| job | string | nil | Job标签值 |
| instance | string | nil | Instance标签值 |

#### 返回值

返回Prometheus数据库对象

### insert

插入指标数据

#### 函数原型

```lua
function db:insert(data, auto_clean)
end
```

#### 参数说明

* data
  数据对象，必须具有encode方法
* auto_clean
  可选，是否在插入后自动清理数据（默认false）

#### 返回值

成功返回true，失败返回nil和错误信息

#### HTTP状态码

- **204**：插入成功（无响应内容）
- 其他：失败

#### 示例

```lua
local pdata = require 'db.prometheus.data'
local pmetric = require 'db.prometheus.metric'

-- 创建指标
local metric = pmetric:new('http_requests_total', {
    method = 'GET',
    status = '200'
}, 'counter', 'Total HTTP requests')

metric:push_value(1, timestamp)

-- 创建数据容器
local data = pdata:new()
data:add_metric(metric)

-- 插入数据库
local ok, err = db:insert(data, true)
```

### insert_metric

插入单个指标

#### 函数原型

```lua
function db:insert_metric(metric, auto_clean)
end
```

#### 参数说明

* metric
  指标对象
* auto_clean
  可选，是否在插入后自动清理数据

#### 返回值

成功返回true，失败返回nil和错误信息

### query

执行即时查询（Instant Query）

#### 函数原型

```lua
function db:query(query, time, timeout)
end
```

#### 参数说明

* query
  PromQL查询字符串
* time
  可选，查询时间点（RFC3339或Unix时间戳）
* timeout
  可选，超时时间（duration字符串，如'30s'）

#### 返回值

成功返回查询结果数据，失败返回nil、错误信息和错误类型

#### 示例

```lua
-- 查询当前值
local result = db:query('up')

-- 查询指定时间点
local result = db:query('up', '2021-01-01T00:00:00Z')
local result = db:query('up', 1609459200)

-- 带超时
local result = db:query('up', nil, '30s')

-- 处理结果
if result then
    print('Result type:', result.resultType)
    for _, item in ipairs(result.result) do
        print('Metric:', item.metric.__name__)
        print('Value:', item.value[2])
    end
end
```

### query_range

执行范围查询（Range Query）

#### 函数原型

```lua
function db:query_range(query, start, end, step, timeout)
end
```

#### 参数说明

* query
  PromQL查询字符串
* start
  起始时间（RFC3339或Unix时间戳）
* end
  结束时间（RFC3339或Unix时间戳）
* step
  查询步长（duration字符串或秒数）
* timeout
  可选，超时时间（duration字符串）

#### 返回值

成功返回查询结果数据，失败返回nil、错误信息和错误类型

#### 示例

```lua
-- 查询过去1小时的数据，每分钟一个点
local end_ts = os.time()
local start_ts = end_ts - 3600

local result = db:query_range(
    'rate(http_requests_total[5m])',
    start_ts,
    end_ts,
    '1m'  -- 或 60
)

-- 处理结果
if result then
    for _, item in ipairs(result.result) do
        print('Metric:', item.metric.__name__)
        for _, value in ipairs(item.values) do
            print('  ', value[1], value[2])
        end
    end
end
```

### query_series

查询匹配的时序列表

#### 函数原型

```lua
function db:query_series(match, start, end)
end
```

#### 参数说明

* match
  时序匹配器（如'{__name__=~".*"}'）
* start
  可选，起始时间
* end
  可选，结束时间

#### 返回值

成功返回时序数组，失败返回nil和错误信息

#### 示例

```lua
-- 查询所有时序
local series = db:query_series('{__name__=~".*"}')

-- 查询特定时序
local series = db:query_series('{__name__="http_requests_total"}')

-- 查询带标签的时序
local series = db:query_series('{job="myapp"}')

-- 处理结果
for _, s in ipairs(series) do
    print('Metric:', s.__name__)
    for k, v in pairs(s) do
        if k ~= '__name__' then
            print('  Label:', k, '=', v)
        end
    end
end
```

### query_labels

查询标签值

#### 函数原型

```lua
function db:query_labels(match, start, end)
end
```

#### 参数说明

* match
  可选，时序匹配器
* start
  可选，起始时间
* end
  可选，结束时间

#### 返回值

成功返回标签名数组，失败返回nil和错误信息

#### 示例

```lua
-- 获取所有标签名
local labels = db:query_labels()

-- 获取特定时序的标签名
local labels = db:query_labels('{__name__="http_requests_total"}')

-- 处理结果
for _, label in ipairs(labels) do
    print('Label:', label)
end
```

## 完整示例

### 应用监控

```lua
local skynet = require 'skynet'
local prom_db = require 'db.prometheus.database'
local pmetric = require 'db.prometheus.metric'
local pdata = require 'db.prometheus.data'

-- 创建连接
local db = prom_db:new({
    host = 'prometheus.example.com',
    port = 9090,
    job = 'freeioe_app',
    instance = 'server01'
})

-- 创建HTTP请求计数器
local function create_http_counter(method, status)
    local name = string.format('http_requests_total', method, status)
    local metric = pmetric:new(name, {
        method = method,
        status = tostring(status)
    }, 'counter', 'Total HTTP requests')

    metric:push_value(1, skynet.time())
    return metric
end

-- 记录HTTP请求
local function record_http_request(method, status)
    local metric = create_http_counter(method, status)

    local data = pdata:new()
    data:add_metric(metric)

    return db:insert_metric(metric, true)
end

-- 查询请求速率
local function get_request_rate()
    local result = db:query('rate(http_requests_total[5m])')

    if result and result.result then
        for _, item in ipairs(result.result) do
            local metric = item.metric
            local value = item.value[2]
            print(string.format('%s %s=%s: %s req/s',
                metric.method, metric.status, value))
        end
    end
end

-- 使用示例
record_http_request('GET', 200)
record_http_request('POST', 201)
record_http_request('GET', 404)

get_request_rate()
```

### 自定义指标

```lua
local pmetric = require 'db.prometheus.metric'

-- 温度指标
local temp = pmetric:new('sensor_temperature_celsius', {
    sensor_id = 'S001',
    location = 'room1'
}, 'gauge', 'Temperature in Celsius')

temp:push_value(25.5, skynet.time())

-- 内存使用指标
local mem = pmetric:new('process_resident_memory_bytes', {
    app_name = 'myapp'
}, 'gauge', 'Resident memory size in bytes')

mem:push_value(1024 * 1024 * 100, skynet.time())  -- 100MB
```

### 批量指标收集

```lua
local pdata = require 'db.prometheus.data'
local pmetric = require 'db.prometheus.metric'

local function collect_metrics()
    local data = pdata:new()

    -- CPU使用率
    local cpu = pmetric:new('cpu_usage_percent', {
        cpu = 'cpu0'
    }, 'gauge', 'CPU usage percentage')
    cpu:push_value(45.2, skynet.time())
    data:add_metric(cpu)

    -- 内存使用率
    local mem = pmetric:new('memory_usage_percent', nil, 'gauge')
    mem:push_value(68.5, skynet.time())
    data:add_metric(mem)

    -- 磁盘使用率
    local disk = pmetric:new('disk_usage_percent', {
        mountpoint = '/'
    }, 'gauge')
    disk:push_value(82.3, skynet.time())
    data:add_metric(disk)

    -- 批量插入
    return db:insert(data, true)
end

collect_metrics()
```

## PromQL查询示例

### 基本查询

```lua
-- 查询即时值
db:query('up')                              -- 所有实例的up状态
db:query('http_requests_total')            -- 总请求数
db:query('node_memory_MemAvailable_bytes') -- 可用内存
```

### 聚合操作

```lua
-- 求和
db:query('sum(http_requests_total)')

-- 平均值
db:query('avg(node_cpu_seconds_total)')

-- 最大值
db:query('max(response_time_seconds)')

-- 最小值
db:query('min(response_time_seconds)')

-- 计数
db:query('count(up)')
```

### 时间函数

```lua
-- 速率（5分钟平均）
db:query('rate(http_requests_total[5m])')

-- 增量
db:query('increase(http_requests_total[1h])')

-- 变化率
db:query('delta(temp_celsius[30m])')
```

### 标签过滤

```lua
-- 精确匹配
db:query('http_requests_total{method="GET"}')

-- 正则匹配
db:query('http_requests_total{method=~"GET|POST"}')

-- 反向匹配
db:query('http_requests_total{status!~"2.."}')
```

### 复杂查询

```lua
-- 查询高错误率
db:query('rate(http_requests_total{status=~"5.."}[5m]) > 0.05')

-- 查询P95延迟
db:query('histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))')

-- 查询CPU使用率Top 10
db:query('topk(10, rate(node_cpu_seconds_total{mode!="idle"}[5m]))')
```

## 错误处理

```lua
-- 插入错误处理
local ok, err = db:insert(data)
if not ok then
    log:error('Prometheus insert failed:', err)
    -- 处理错误
end

-- 查询错误处理
local result, err, err_type = db:query('invalid_query')
if not result then
    log:error('Query failed:', err, 'Type:', err_type)
    -- err_type可能为：'bad_data', 'execution', 'timeout'等
end
```

## 性能建议

1. **批量写入**：使用数据容器批量插入多个指标
2. **基数控制**：避免高基数标签（如user_id）
3. **查询优化**：使用合理的时间范围和步长
4. **标签设计**：合理设计标签体系
5. **指标类型**：选择正确的指标类型（counter/gauge/histogram/summary）

## 与Pushgateway集成

如果需要使用Pushgateway：

```lua
local db = prom_db:new({
    host = 'pushgateway.example.com',
    port = 9091,
    url = '/metrics/job/myapp/instance/localhost'
})
```
