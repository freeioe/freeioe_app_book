---

# SiriDB数据库模块 (db.siridb.database)

本模块提供SiriDB数据库的客户端接口，支持数据插入、查询和数据库管理操作。

## 使用方法

```lua
local siridb_db = require 'db.siridb.database'

-- 创建数据库连接
local db = siridb_db:new({
    host = '127.0.0.1',
    port = 9020,
    username = 'iris',
    password = 'siri',
    time_precision = 'ms',
    timeout = 5000
}, 'mydb')

-- 插入数据
local ok, err = db:insert(data)

-- 查询数据
local result, err = db:query('SELECT * FROM temperature')

-- 执行SQL
local result, err = db:exec('CREATE DATABASE test')
```

## 接口说明

### new

创建SiriDB数据库对象

#### 函数原型

```lua
function siridb_db:new(options, dbname)
end
```

#### 参数说明

* options
  连接选项表

| 选项 | 类型 | 默认值 | 说明 |
| :--- | :--- | :--- | :--- |
| host | string | '127.0.0.1' | 数据库主机地址 |
| port | number | 9020 | 数据库端口 |
| ssl | boolean | false | 是否使用SSL/TLS |
| username | string | 'iris' | 用户名 |
| password | string | 'siri' | 密码 |
| timeout | number | nil | 请求超时（毫秒） |
| time_precision | string | 'ms' | 时间精度（s/ms/us/ns） |
| db | string | nil | 数据库名称 |

* dbname
  可选，数据库名称（优先级高于options.db）

#### 返回值

返回SiriDB数据库对象

### time_precision

获取时间精度设置

#### 函数原型

```lua
function db:time_precision()
end
```

#### 返回值

返回时间精度字符串（'s', 'ms', 'us', 'ns'）

### dbname

获取数据库名称

#### 函数原型

```lua
function db:dbname()
end
```

#### 返回值

返回数据库名称字符串

### insert

插入数据到数据库

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

#### 示例

```lua
local sdata = require 'db.siridb.data'
local sseries = require 'db.siridb.series'

-- 创建时序数据
local series = sseries:new('temperature', 'float')
series:push_value(25.5, skynet.time())

-- 创建数据容器
local data = sdata:new()
data:add_series('temperature', series)

-- 插入数据库
local ok, err = db:insert(data, true)  -- 插入后自动清理
```

### insert_series

插入单个时序数据

#### 函数原型

```lua
function db:insert_series(series, auto_clean)
end
```

#### 参数说明

* series
  时序数据对象
* auto_clean
  可选，是否在插入后自动清理数据

#### 返回值

成功返回true，失败返回nil和错误信息

#### 示例

```lua
local sseries = require 'db.siridb.series'

local series = sseries:new('humidity', 'float')
series:push_value(65.2, skynet.time())

db:insert_series(series)
```

### query

执行SiriDB查询

#### 函数原型

```lua
function db:query(query, time_precision)
end
```

#### 参数说明

* query
  SiriDB查询语句字符串
* time_precision
  可选，返回数据的时间精度（默认使用连接时的设置）

#### 返回值

成功返回查询结果表，失败返回nil、错误信息和状态码

#### 注意事项

> 时间范围为[start, end)，即包含start但不包含end。如需包含end，应增加1ms。

#### 示例

```lua
-- 查询所有数据
local result = db:query('SELECT * FROM temperature')

-- 查询时间范围
local start_time = 1609459200  -- 2021-01-01 00:00:00
local end_time = 1609545600    -- 2021-01-02 00:00:00
local result = db:query(string.format(
    'SELECT * FROM temperature AFTER %d BEFORE %d',
    start_time, end_time
))

-- 聚合查询
local result = db:query('SELECT MEAN(value) FROM temperature')
```

### exec

执行SQL命令

#### 函数原型

```lua
function db:exec(sql)
end
```

#### 参数说明

* sql
  SQL命令字符串

#### 返回值

成功返回执行结果表，失败返回nil、错误信息和状态码

#### 示例

```lua
-- 创建数据库
local result = db:exec('CREATE DATABASE mydb')

-- 创建用户
local result = db:exec("CREATE USER john WITH PASSWORD 'secret'")

-- 授予权限
local result = db:exec('GRANT ALL ON mydb TO john')

-- 删除数据
local result = db:exec('DELETE FROM temperature WHERE time < 1609459200')
```

## 数据压缩

本模块支持使用qpack压缩以提高性能：

```lua
-- 如果安装了qpack模块，将自动启用压缩
-- qpack比JSON更高效，减少网络传输和序列化开销
```

## 完整示例

```lua
local skynet = require 'skynet'
local siridb_db = require 'db.siridb.database'
local sseries = require 'db.siridb.series'
local sdata = require 'db.siridb.data'

-- 创建连接
local db = siridb_db:new({
    host = '192.168.1.100',
    port = 9020,
    username = 'admin',
    password = 'password',
    time_precision = 'ms'
}, 'production')

-- 插入温度数据
local function insert_temperature(device_id, value, timestamp)
    local series_name = string.format('temp_device_%s', device_id)

    -- 创建时序
    local series = sseries:new(series_name, 'float')
    series:push_value(value, timestamp or skynet.time())

    -- 插入
    return db:insert_series(series, true)
end

-- 批量插入
local function batch_insert(readings)
    local data = sdata:new()

    for device_id, value in pairs(readings) do
        local series_name = string.format('temp_device_%s', device_id)
        local series = sseries:new(series_name, 'float')
        series:push_value(value, skynet.time())
        data:add_series(series_name, series)
    end

    return db:insert(data, true)
end

-- 查询最新数据
local function get_latest_temperature(device_id)
    local series_name = string.format('temp_device_%s', device_id)
    local query = string.format(
        'SELECT * FROM "%s" LIMIT 1',
        series_name
    )
    return db:query(query)
end

-- 查询时间范围平均值
local function get_average_temperature(start_ts, end_ts)
    local query = string.format(
        'SELECT MEAN(value) FROM temp_device_1 AFTER %d BEFORE %d',
        start_ts, end_ts + 1  -- +1ms以包含end时间点
    )
    return db:query(query)
end

-- 使用示例
insert_temperature('sensor1', 25.3)
insert_temperature('sensor2', 24.8)

batch_readings = {
    sensor1 = 25.5,
    sensor2 = 24.9,
    sensor3 = 26.1
}
batch_insert(batch_readings)

local latest = get_latest_temperature('sensor1')
local avg = get_average_temperature(start_time, end_time)
```

## 错误处理

```lua
local ok, err = db:insert(data)
if not ok then
    log:error('SiriDB insert failed:', err)
    -- 检查错误类型并处理
    if err:match('database not found') then
        -- 创建数据库
    elseif err:match('authentication failed') then
        -- 重新认证
    else
        -- 其他错误
    end
end
```

## 常见查询示例

### 基本查询

```lua
-- 选择所有数据
'SELECT * FROM series_name'

-- 选择最新N条
'SELECT * FROM series_name LIMIT 10'

-- 时间范围查询
'SELECT * FROM series_name AFTER 1609459200 BEFORE 1609545600'
```

### 聚合查询

```lua
-- 平均值
'SELECT MEAN(value) FROM series_name'

-- 最大值
'SELECT MAX(value) FROM series_name'

-- 最小值
'SELECT MIN(value) FROM series_name'

-- 求和
'SELECT SUM(value) FROM series_name'

-- 计数
'SELECT COUNT(value) FROM series_name'
```

### 分组查询

```lua
-- 按时间分组
'SELECT MEAN(value) FROM series_name GROUP BY 1h'

-- 按时间粒度分组
'SELECT MEAN(value) FROM series_name GROUP BY 5m'
```

## 性能建议

1. **批量写入**：使用data对象批量插入多个时序
2. **时间范围限制**：始终在查询中指定时间范围
3. **索引使用**：合理设计时序名称以提高查询效率
4. **连接复用**：保持数据库连接而非频繁创建
