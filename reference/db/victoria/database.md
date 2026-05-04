---

# VictoriaMetrics数据库模块 (db.victoria.database)

本模块提供VictoriaMetrics数据库的客户端接口。VictoriaMetrics是一个高性能的时序数据库，与Prometheus兼容，提供更好的性能和压缩率。

## 特点

- **高性能**：比Prometheus快10-20倍
- **高压缩率**：数据存储占用空间更少
- **Prometheus兼容**：支持PromQL查询语言
- **简单部署**：无需外部依赖，单二进制文件
- **长期存储**：适合长期数据保留

## 使用方法

```lua
local victoria_db = require 'db.victoria.database'

-- 创建数据库连接
local db = victoria_db:new({
    host = '127.0.0.1',
    port = 8428,
    username = 'admin',
    password = 'password'
})

-- 导出数据
local result, err = db:export('{__name__=~".*"}', start_time, end_time)
```

## 接口说明

### new

创建VictoriaMetrics数据库对象

#### 函数原型

```lua
function victoria_db:new(options)
end
```

#### 参数说明

* options
  连接选项表

| 选项 | 类型 | 默认值 | 说明 |
| :--- | :--- | :--- | :--- |
| host | string | '127.0.0.1' | VictoriaMetrics服务器地址 |
| port | number | 8428 | HTTP端口（单机版默认8428） |
| ssl | boolean | false | 是否使用SSL/TLS |
| username | string | nil | 认证用户名（可选） |
| password | string | nil | 认证密码（可选） |
| timeout | number | nil | 请求超时（毫秒，可选） |

#### 返回值

返回VictoriaMetrics数据库对象

### export

导出指定时间范围的时序数据

#### 函数原型

```lua
function db:export(match, start, end, max_rows_per_line)
end
```

#### 参数说明

* match
  时序选择器，用于筛选要导出的时序
* start
  起始时间（Unix时间戳，秒）
* end
  结束时间（Unix时间戳，秒）
* max_rows_per_line
  可选，每行最大数据点数

#### 返回值

成功返回导出数据数组，失败返回nil和错误信息

#### 导出数据格式

```lua
{
    {
        metric = {
            __name__ = 'metric_name',
            label1 = 'value1',
            label2 = 'value2'
        },
        value = '123.456',
        timestamp = 1609459200
    },
    ...
}
```

## 时序选择器

VictoriaMetrics支持Prometheus风格的时序选择器：

### 基本选择器

```lua
-- 选择所有时序
'{__name__=~".*"}'

-- 选择特定指标
'{__name__="http_requests_total"}'

-- 精确匹配标签值
'{job="prometheus"}'

-- 正则匹配标签值
'{__name__=~"http_.*"}'

-- 反向匹配
'{__name__!~"debug_.*"}'
```

### 组合选择器

```lua
-- 多个标签条件
'{__name__="http_requests_total",method="GET",status="200"}'

-- 或逻辑（需要多次查询）
-- 分别查询 '{method="GET"}' 和 '{method="POST"}'
```

### 高级选择器

```lua
-- 匹配多个标签值
'{method=~"GET|POST|PUT"}'

-- 排除特定值
'{status!~"2.."}'  -- 排除2xx状态码

-- 复杂正则
'{instance=~"server-\\d+"}'
```

## 完整示例

### 基本数据导出

```lua
local victoria_db = require 'db.victoria.database'

-- 创建连接
local db = victoria_db:new({
    host = 'victoria-metrics.example.com',
    port = 8428
})

-- 导出最近1小时数据
local end_ts = os.time()
local start_ts = end_ts - 3600

local result, err = db:export('{__name__=~".*"}', start_ts, end_ts)

if result then
    print('Exported', #result, 'data points')
    for _, item in ipairs(result) do
        local metric = item.metric.__name__
        local value = item.value
        local ts = item.timestamp

        print(string.format('%s: %s @ %d', metric, value, ts))
    end
else
    print('Export failed:', err)
end
```

### 导出特定指标

```lua
-- 导出HTTP请求指标
local result = db:export(
    '{__name__="http_requests_total"}',
    start_time,
    end_time
)

if result then
    -- 按方法分组统计
    local stats = {}
    for _, item in ipairs(result) do
        local method = item.metric.method
        stats[method] = (stats[method] or 0) + tonumber(item.value)
    end

    for method, count in pairs(stats) do
        print(string.format('%s: %d requests', method, count))
    end
end
```

### 分批导出大数据

```lua
-- 分批导出避免超时
local function export_batches(match, start_ts, end_ts, batch_hours)
    batch_hours = batch_hours or 1
    local batch_size = batch_hours * 3600

    local all_data = {}
    local current = start_ts

    while current < end_ts do
        local batch_end = math.min(current + batch_size, end_ts)

        print(string.format('Exporting %s to %s',
            os.date('%Y-%m-%d %H:%M', current),
            os.date('%Y-%m-%d %H:%M', batch_end)))

        local data, err = db:export(match, current, batch_end)
        if not data then
            print('Batch failed:', err)
            break
        end

        -- 合并数据
        for _, item in ipairs(data) do
            table.insert(all_data, item)
        end

        current = batch_end
    end

    return all_data
end

-- 使用：每批导出2小时数据
local data = export_batches('{__name__=~".*"}', start_ts, end_ts, 2)
```

### 导出到文件

```lua
-- 导出并保存为JSON
local function export_to_file(filename, match, start_ts, end_ts)
    local data, err = db:export(match, start_ts, end_ts)
    if not data then
        return nil, err
    end

    local cjson = require 'cjson'
    local file = io.open(filename, 'w')
    if not file then
        return nil, 'Cannot open file'
    end

    file:write(cjson.encode(data))
    file:close()

    return true
end

-- 导出为CSV
local function export_to_csv(filename, match, start_ts, end_ts)
    local data, err = db:export(match, start_ts, end_ts)
    if not data then
        return nil, err
    end

    local file = io.open(filename, 'w')
    if not file then
        return nil, 'Cannot open file'
    end

    -- CSV头
    file:write('timestamp,metric,labels,value\n')

    -- 数据行
    for _, item in ipairs(data) do
        local metric_name = item.metric.__name__

        -- 构建标签字符串
        local labels = {}
        for k, v in pairs(item.metric) do
            if k ~= '__name__' then
                table.insert(labels, k .. '=' .. v)
            end
        end

        local line = string.format('%d,%s,%s,%s\n',
            item.timestamp,
            metric_name,
            table.concat(labels, ','),
            item.value)

        file:write(line)
    end

    file:close()
    return true
end

-- 使用
export_to_csv('metrics.csv', '{__name__="temperature"}', start_ts, end_ts)
```

## 数据分析和处理

### 数据统计

```lua
local function analyze_metrics(data)
    local stats = {
        count = #data,
        metrics = {},
        min_value = math.huge,
        max_value = -math.huge,
        sum = 0
    }

    for _, item in ipairs(data) do
        local value = tonumber(item.value)
        local name = item.metric.__name__

        -- 值统计
        stats.min_value = math.min(stats.min_value, value)
        stats.max_value = math.max(stats.max_value, value)
        stats.sum = stats.sum + value

        -- 指标计数
        stats.metrics[name] = (stats.metrics[name] or 0) + 1
    end

    stats.avg_value = stats.sum / stats.count
    return stats
end

local data, err = db:export('{__name__="temperature"}', start_ts, end_ts)
local stats = analyze_metrics(data)
print('Statistics:', cjson.encode(stats))
```

### 按标签分组

```lua
local function group_by_label(data, label_key)
    local groups = {}

    for _, item in ipairs(data) do
        local label_value = item.metric[label_key]

        if not groups[label_value] then
            groups[label_value] = {}
        end

        table.insert(groups[label_value], item)
    end

    return groups
end

local data = db:export('{__name__="temperature"}', start_ts, end_ts)
local by_location = group_by_label(data, 'location')

for location, items in pairs(by_location) do
    print(location, ':', #items, 'data points')
end
```

## 性能优化

### 减少数据量

```lua
-- 使用更精确的选择器
'{__name__="http_requests_total",status="500"}'  -- 好
'{__name__=~"http_.*"}'                             -- 差

-- 缩短时间范围
local result = db:export(match, end_ts - 3600, end_ts)  -- 好
local result = db:export(match, start_ts, end_ts)         -- 差

-- 添加更多标签限制
'{job="myapp",instance="server01"}'  -- 好
'{job="myapp"}'                     -- 差
```

### 分批处理

```lua
-- 大数据集分批导出
local batch_size = 3600  -- 每批1小时
for current = start_ts, current < end_ts, current + batch_size do
    local batch_end = math.min(current + batch_size, end_ts)
    local data = db:export(match, current, batch_end)
    -- 处理数据...
end
```

## 与其他系统集成

### 导出到SiriDB

```lua
local siridb_db = require 'db.siridb.database'
local sseries = require 'db.siridb.series'
local sdata = require 'db.siridb.data'

local function migrate_to_siridb(vm_db, siri_db, match, start_ts, end_ts)
    -- 从VictoriaMetrics导出
    local data, err = vm_db:export(match, start_ts, end_ts)
    if not data then
        return nil, err
    end

    -- 转换为SiriDB格式
    local siri_data = sdata:new()
    local series_map = {}

    for _, item in ipairs(data) do
        local name = item.metric.__name__

        if not series_map[name] then
            series_map[name] = sseries:new(name, 'float')
        end

        series_map[name]:push_value(item.value, item.timestamp)
    end

    -- 添加到数据容器
    for name, series in pairs(series_map) do
        siri_data:add_series(name, series)
    end

    -- 插入SiriDB
    return siri_db:insert(siri_data, true)
end
```

### 导出到Prometheus

VictoriaMetrics与Prometheus兼容，可以使用相同的工具：

```lua
-- VictoriaMetrics可以作为Prometheus的替代品
-- 无需修改客户端代码

local victoria_db = victoria_db:new({
    host = 'victoria-metrics',
    port = 8428
})

-- 使用相同的PromQL查询
local result = victoria_db:export('up', start_ts, end_ts)
```

## 错误处理

```lua
local result, err = db:export(match, start_ts, end_ts)

if not result then
    -- 处理错误
    if err:match('timeout') then
        -- 超时，减少时间范围
        print('Timeout, reducing time range')
        result = db:export(match, start_ts, (start_ts + end_ts) / 2)
    elseif err:match('not found') then
        -- 没有匹配的时序
        print('No metrics match selector')
        result = {}
    else
        -- 其他错误
        print('Export failed:', err)
    end
end
```

## 部署配置

### 单机模式

```lua
local db = victoria_db:new({
    host = 'localhost',
    port = 8428  -- 单机版默认端口
})
```

### 集群模式

```lua
-- vmselect端口
local db = victoria_db:new({
    host = 'vmcluster.example.com',
    port = 8481
})

-- 或直接连接vmstorage
local db = victoria_db:new({
    host = 'vmstorage.example.com',
    port = 8400
})
```

### 使用认证

```lua
local db = victoria_db:new({
    host = 'victoria-metrics.example.com',
    port = 8428,
    username = 'admin',
    password = 'secret_password'
})
```

## 最佳实践

1. **查询优化**：
   - 使用精确的选择器
   - 限制时间范围
   - 避免高基数标签

2. **分批导出**：
   - 大数据集分批处理
   - 监控导出时间
   - 保存中间结果

3. **错误处理**：
   - 检查返回值
   - 处理超时
   - 验证数据完整性

4. **性能监控**：
   - 监控导出耗时
   - 控制数据量
   - 使用缓存

## VictoriaMetrics vs Prometheus

| 特性 | VictoriaMetrics | Prometheus |
| :--- | :--- | :--- |
| 性能 | 快10-20倍 | 基准 |
| 存储空间 | 减少10倍 | 基准 |
| 兼容性 | Prometheus兼容 | - |
| 部署 | 单二进制，简单 | 需要配置 |
| 长期存储 | 支持 | 需要Thanos/Cortex |
| 查询语言 | MetricsQL, PromQL | PromQL |

选择VictoriaMetrics的场景：
- 需要长期数据存储
- 大规模数据采集
- 资源受限环境
- Prometheus兼容性需求
