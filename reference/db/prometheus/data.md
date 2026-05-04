---

# Prometheus数据容器模块 (db.prometheus.data)

本模块提供Prometheus数据容器，用于组织和编码多个指标（Metric）的数据。

## 使用方法

```lua
local pdata = require 'db.prometheus.data'
local pmetric = require 'db.prometheus.metric'

-- 创建数据容器
local data = pdata:new()

-- 创建指标
local metric1 = pmetric:new('metric1', nil, 'gauge')
metric1:push_value(100, timestamp)

local metric2 = pmetric:new('metric2', nil, 'counter')
metric2:push_value(1, timestamp)

-- 添加到容器
data:add_metric(metric1)
data:add_metric(metric2)

-- 编码为Prometheus文本格式
local text = data:encode(true)
```

## 接口说明

### new

创建数据容器对象

#### 函数原型

```lua
function pdata:new()
end
```

#### 返回值

返回数据容器对象

### add_metric

添加指标到容器

#### 函数原型

```lua
function data:add_metric(metric)
end
```

#### 参数说明

* metric
  指标对象

#### 注意事项

- 同一指标可以多次添加（每个会生成一行）
- 不同指标的添加顺序会影响输出顺序

### encode

编码容器中所有指标的数据

#### 函数原型

```lua
function data:encode(auto_clean)
end
```

#### 参数说明

* auto_clean
  可选，编码后是否自动清空各指标的数据（默认false）

#### 返回值

返回Prometheus文本格式的字符串，可直接发送到Pushgateway或通过`/metrics`端点暴露

#### 输出格式

```
# TYPE metric1 gauge
# HELP metric1 Metric description
metric1{label1="value1"} 100 1609459200000
metric1{label1="value2"} 200 1609459260000
# TYPE metric2 counter
# HELP metric2 Another metric
metric2 1 1609459200000
```

## 完整示例

### 多指标收集

```lua
local pdata = require 'db.prometheus.data'
local pmetric = require 'db.prometheus.metric'

-- 创建容器
local data = pdata:new()

-- 添加各种类型指标

-- CPU使用率
local cpu = pmetric:new('cpu_usage_percent', {
    cpu = 'cpu0',
    mode = 'user'
}, 'gauge', 'CPU usage percentage')
cpu:push_value(45.2, timestamp)
data:add_metric(cpu)

-- 内存使用
local mem = pmetric:new('memory_usage_bytes', {
    app = 'myapp'
}, 'gauge', 'Memory usage in bytes')
mem:push_value(1024 * 1024 * 100, timestamp)
data:add_metric(mem)

-- 请求计数
local requests = pmetric:new('http_requests_total', {
    method = 'GET',
    status = '200'
}, 'counter', 'Total HTTP requests')
requests:push_value(1, timestamp)
data:add_metric(requests)

-- 编码
local prometheus_text = data:encode(true)
print(prometheus_text)
```

### 动态指标添加

```lua
local data = pdata:new()

-- 动态添加多个传感器的指标
local sensor_readings = {
    {id='S001', value=25.5},
    {id='S002', value=24.8},
    {id='S003', value=26.1},
}

for _, reading in ipairs(sensor_readings) do
    local metric = pmetric:new('sensor_temperature_celsius', {
        sensor_id = reading.id,
        location = 'room1'
    }, 'gauge', 'Temperature sensor reading')

    metric:push_value(reading.value, timestamp)
    data:add_metric(metric)
end

-- 一次性发送所有传感器数据
local text = data:encode(true)
```

### 指标分组

```lua
local function collect_app_metrics(app_name)
    local data = pdata:new()

    -- 应用信息
    local info = pmetric:new('app_info', {
        name = app_name,
        version = '1.0.0'
    }, 'gauge', 'Application information')
    info:push_value(1, timestamp)
    data:add_metric(info)

    -- 性能指标
    local perf = pmetric:new('app_performance_score', {
        name = app_name
    }, 'gauge', 'Application performance score')
    perf:push_value(95.5, timestamp)
    data:add_metric(perf)

    return data
end

local data = collect_app_metrics('myapp')
db:insert(data, true)
```

### 批量历史数据

```lua
local data = pdata:new()

-- 创建一个指标，添加多个历史数据点
local metric = pmetric:new('temperature', {
    sensor_id = 'S001'
}, 'gauge')

-- 添加过去1小时每分钟的数据
for i = 0, 59 do
    local ts = timestamp - i * 60
    local value = 20 + math.random() * 10
    metric:push_value(value, ts)
end

data:add_metric(metric)

-- 编码生成多行输出
local text = data:encode(true)
local lines = {}
for line in text:gmatch('[^\r\n]+') do
    table.insert(lines, line)
end
print('Total lines:', #lines)
```

## HTTP端点暴露

### 创建/metrics端点

```lua
local pdata = require 'db.prometheus.data'
local pmetric = require 'db.prometheus.metric'

-- 获取所有指标
local function get_metrics()
    local data = pdata:new()

    -- 添加系统指标
    local cpu = pmetric:new('system_cpu_usage', nil, 'gauge')
    cpu:push_value(get_cpu_usage(), skynet.time())
    data:add_metric(cpu)

    local mem = pmetric:new('system_memory_usage', nil, 'gauge')
    mem:push_value(get_memory_usage(), skynet.time())
    data:add_metric(mem)

    -- 编码为Prometheus格式
    return data:encode()
end

-- 在HTTP服务中暴露
app:get('/metrics', function()
    local text = get_metrics()
    return text, 'text/plain; version=0.0.4; charset=utf-8'
end)
```

### 与Prometheus集成

```lua
-- Prometheus配置示例
-- prometheus.yml:
-- scrape_configs:
--   - job_name: 'freeioe_app'
--     scrape_interval: 15s
--     static_configs:
--       - targets: ['localhost:8080']
--         labels:
--           app: 'freeioe'

-- 应用提供/metrics端点
local function handle_metrics_request()
    local data = collect_all_metrics()
    local text = data:encode()

    -- 设置响应头
    set_header('Content-Type', 'text/plain; version=0.0.4')
    return text
end
```

## 数据验证

### 检查编码结果

```lua
local data = pdata:new()
-- ... 添加指标 ...

local text = data:encode()

-- 验证格式
local function validate_prometheus_text(text)
    -- 检查空行分隔
    local parts = {}
    for part in text:gmatch('[^\n]+') do
        table.insert(parts, part)
    end

    -- 每个指标应该有TYPE行（可选HELP）和数据行
    local i = 1
    while i <= #parts do
        if parts[i]:match('^# TYPE') then
            -- 有TYPE定义
            i = i + 1
            if parts[i]:match('^# HELP') then
                -- 有HELP定义
                i = i + 1
            end
            -- 应该有数据行
            if not parts[i]:match('^%a') then
                return false, 'Missing data line after TYPE'
            end
        end
        i = i + 1
    end

    return true
end

local ok, err = validate_prometheus_text(text)
if not ok then
    log:error('Invalid Prometheus text:', err)
end
```

## 性能优化

### 避免频繁编码

```lua
-- 低效：每次添加都编码
local data = pdata:new()
for i = 1, 1000 do
    local m = pmetric:new('test', nil, 'gauge')
    m:push_value(i, timestamp)
    data:add_metric(m)
    db:insert(data)  -- 每次都插入
end

-- 高效：批量添加后一次性编码
local data = pdata:new()
for i = 1, 1000 do
    local m = pmetric:new('test', nil, 'gauge')
    m:push_value(i, timestamp)
    data:add_metric(m)
end
db:insert(data, true)  -- 一次性插入
```

### 控制指标数量

```lua
local function collect_metrics()
    local data = pdata:new()

    -- 添加关键指标
    add_critical_metrics(data)

    -- 根据配置添加详细指标
    if config.detailed_metrics then
        add_detailed_metrics(data)
    end

    return data
end
```

## 错误处理

```lua
local data = pdata:new()

-- 安全添加指标
local function safe_add_metric(data, name, labels, typ, value, timestamp)
    local ok, err = pcall(function()
        local metric = pmetric:new(name, labels, typ)
        metric:push_value(value, timestamp)
        data:add_metric(metric)
    end)

    if not ok then
        log.error('Failed to add metric:', name, err)
        return false
    end

    return true
end

-- 使用
safe_add_metric(data, 'test', {key='value'}, 'gauge', 100, timestamp)
```

## 与Pushgateway集成

```lua
local function push_to_pushgateway(data, url, job, instance)
    local http = require 'http.restful'
    local text = data:encode()

    -- 构建URL
    local push_url = string.format('%s/metrics/job/%s', url, job)
    if instance then
        push_url = push_url .. '/instance/' .. instance
    end

    -- 推送数据
    local client = http:new(url)
    local status, body = client:post(push_url, nil, text, nil, {
        ['Content-Type'] = 'text/plain; version=0.0.4; charset=utf-8'
    })

    return status == 200 or status == 202
end

-- 使用
local data = pdata:new()
-- ... 添加指标 ...

push_to_pushgateway(data, 'http://pushgateway:9091', 'myapp', 'host1')
```

## 最佳实践

1. **批量操作**：收集多个指标后一次性编码
2. **自动清理**：使用auto_clean参数释放内存
3. **指标组织**：相关指标一起收集和发送
4. **错误处理**：验证编码结果
5. **性能监控**：控制指标数量和标签基数
