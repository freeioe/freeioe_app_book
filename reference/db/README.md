---

# 数据库模块 (db)

本模块提供了多种时序数据库的客户端接口，用于存储和查询时间序列数据。

## 支持的数据库

| 数据库 | 说明 | 目录 |
| :--- | :--- | :--- |
| **SiriDB** | 开源时序数据库，高性能分布式 | [siridb/](siridb/) |
| **Prometheus** | 云原生监控系统和时序数据库 | [prometheus/](prometheus/) |
| **VictoriaMetrics** | 高性能时序数据库，Prometheus兼容 | [victoria/](victoria/) |
| **InfluxDB** | 开源时序数据库 | [influxdb/](influxdb/) |

## 模块结构

```
db/
├── siridb/          # SiriDB客户端
│   ├── client.lua   # 管理客户端
│   ├── database.lua # 数据库操作
│   ├── series.lua   # 时序数据
│   └── data.lua     # 数据容器
├── prometheus/      # Prometheus客户端
│   ├── database.lua # 数据库操作
│   ├── metric.lua   # 指标数据
│   └── data.lua     # 数据容器
├── victoria/        # VictoriaMetrics客户端
│   └── database.lua # 数据库操作
└── influxdb/        # InfluxDB客户端
    ├── query.lua    # 查询接口
    └── lineproto.lua # 行协议编码
```

## 使用方法

### SiriDB

```lua
local siridb_db = require 'db.siridb.database'
local sseries = require 'db.siridb.series'
local sdata = require 'db.siridb.data'

-- 连接数据库
local db = siridb_db:new({
    host = '127.0.0.1',
    port = 9020,
    username = 'iris',
    password = 'siri',
    time_precision = 'ms'
}, 'mydb')

-- 插入数据
local series = sseries:new('temperature', 'float')
series:push_value(25.5, os.time())

local data = sdata:new()
data:add_series('temperature', series)
db:insert(data)

-- 查询数据
local result = db:query('SELECT * FROM temperature')
```

### Prometheus

```lua
local prom_db = require 'db.prometheus.database'
local pmetric = require 'db.prometheus.metric'

-- 连接数据库
local db = prom_db:new({
    host = '127.0.0.1',
    port = 9090,
    job = 'myapp',
    instance = 'localhost'
})

-- 插入指标
local metric = pmetric:new('http_requests_total', {
    method = 'GET',
    status = '200'
}, 'counter', 'Total HTTP requests')

metric:push_value(1, os.time())
db:insert_metric(metric)

-- 查询数据
local result = db:query('http_requests_total')
```

### VictoriaMetrics

```lua
local victoria_db = require 'db.victoria.database'

-- 连接数据库
local db = victoria_db:new({
    host = '127.0.0.1',
    port = 8428,
    username = 'admin',
    password = 'password'
})

-- 导出数据
local result = db:export('{__name__=~".*"}', start_time, end_time)
```

## 通用概念

### 时间精度

所有数据库都支持多种时间精度：

| 精度 | 说明 | 值 |
| :--- | :--- | :--- |
| s | 秒 | 1 |
| ms | 毫秒 | 1000 |
| us | 微秒 | 1000000 |
| ns | 纳秒 | 1000000000 |

### 数据类型

支持的值类型：

| 类型 | 说明 | 示例 |
| :--- | :--- | :--- |
| int | 整数 | 42 |
| float | 浮点数 | 3.14 |
| string | 字符串 | "error" |

### 连接选项

所有数据库客户端都支持以下选项：

```lua
{
    host = '127.0.0.1',      -- 数据库主机
    port = 9090,             -- 端口
    ssl = false,             -- 是否使用SSL/TLS
    username = 'user',       -- 用户名（可选）
    password = 'pass',       -- 密码（可选）
    timeout = 5000,          -- 请求超时（毫秒）
    time_precision = 'ms'    -- 时间精度
}
```

## 数据写入模式

### 单点写入

```lua
local metric = pmetric:new('metric_name', {label='value'})
metric:push_value(100, timestamp)
db:insert_metric(metric)
```

### 批量写入

```lua
local data = sdata:new()
data:add_series('series1', series1)
data:add_series('series2', series2)
db:insert(data)
```

## 选择合适的数据库

### SiriDB

**优势**：
- 分布式架构
- 高压缩比
- 支持SQL查询
- 自动数据分片

**适用场景**：
- 大规模时序数据
- 需要分布式部署
- 需要SQL兼容性

### Prometheus

**优势**：
- 云原生集成
- 强大的查询语言（PromQL）
- 服务发现支持
- 告警规则集成

**适用场景**：
- 监控和告警
- Kubernetes环境
- 云原生应用

### VictoriaMetrics

**优势**：
- 高性能（比Prometheus快10倍）
- 更低的资源消耗
- Prometheus兼容
- 支持长期存储

**适用场景**：
- 高基数指标
- 长期数据存储
- Prometheus替代

### InfluxDB

**优势**：
- 成熟稳定
- 类SQL查询
- 丰富的生态
- 内置数据降采样

**适用场景**：
- IoT数据存储
- 传统监控
- 实时分析

## 性能建议

### 写入优化

1. **批量写入**：尽量使用批量接口而非单点写入
2. **数据压缩**：启用数据库的压缩功能
3. **连接池**：复用数据库连接

### 查询优化

1. **时间范围限制**：始终指定时间范围
2. **数据降采样**：对历史数据使用降采样
3. **索引利用**：合理使用标签索引

## 错误处理

```lua
local ok, err = db:insert(data)
if not ok then
    log:error('Insert failed:', err)
    -- 处理错误
end
```

常见错误：
- **连接失败**：检查主机和端口
- **认证失败**：验证用户名和密码
- **数据库不存在**：先创建数据库
- **查询语法错误**：检查查询语句

## 安全建议

1. **使用SSL/TLS**：生产环境启用加密连接
2. **认证授权**：设置强密码和适当的权限
3. **网络隔离**：数据库服务器在内网
4. **定期备份**：制定备份策略
