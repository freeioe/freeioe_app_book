---

# InfluxDB查询模块 (db.influxdb.query)

本模块提供InfluxDB数据库的查询接口，基于InfluxDB的HTTP API。

## 使用方法

```lua
local query = require 'db.influxdb.query'

-- 创建查询对象
local q = query:new({
    scheme = 'http',
    host = 'localhost',
    port = 8086,
    username = 'username',
    password = 'password',
    database = 'mydb'
})

-- 执行查询
local result, err = q:query('SELECT * FROM measurement WHERE time > now() - 1h')
```

## 接口说明

### new

创建InfluxDB查询对象

#### 函数原型

```lua
function query:new(options)
end
```

#### 参数说明

* options
  连接选项表

| 选项 | 类型 | 默认值 | 说明 |
| :--- | :--- | :--- | :--- |
| scheme | string | 'http' | 协议类型（http/https） |
| host | string | 'localhost' | InfluxDB主机地址 |
| port | number | 8086 | InfluxDB端口 |
| username | string | nil | 用户名 |
| password | string | nil | 密码 |
| database | string | nil | 数据库名 |
| timeout | number | nil | 请求超时（毫秒） |

#### 返回值

返回InfluxDB查询对象

### query

执行InfluxQL查询

#### 函数原型

```lua
function query:query(query_string)
end
```

#### 参数说明

* query_string
  InfluxQL查询语句字符串

#### 返回值

成功返回查询结果表（JSON解码后），失败返回nil和错误信息

#### 查询结果格式

```lua
{
    results = {
        {
            statement_id = 0,
            series = {
                {
                    name = "measurement_name",
                    columns = {"time", "value", "tag1"},
                    values = {
                        {timestamp, value, tag_value},
                        {timestamp, value, tag_value},
                        ...
                    }
                }
            }
        }
    }
}
```

## 完整示例

### 基本查询

```lua
local query = require 'db.influxdb.query'

local q = query:new({
    host = 'localhost',
    port = 8086,
    database = 'sensors'
})

-- 查询所有数据
local result = q:query('SELECT * FROM temperature')

-- 查询指定时间范围
local result = q:query('SELECT * FROM temperature WHERE time > now() - 1h')

-- 查询并限制结果数量
local result = q:query('SELECT * FROM temperature LIMIT 100')
```

### 聚合查询

```lua
-- 计算平均值
local result = q:query('SELECT MEAN(value) FROM temperature')

-- 计算最大值
local result = q:query('SELECT MAX(value) FROM temperature')

-- 计算最小值
local result = q:query('SELECT MIN(value) FROM temperature')

-- 计数
local result = q:query('SELECT COUNT(value) FROM temperature')

-- 求和
local result = q:query('SELECT SUM(value) FROM temperature')
```

### 时间范围查询

```lua
-- 查询最近1小时
local result = q:query('SELECT * FROM temperature WHERE time > now() - 1h')

-- 查询最近24小时
local result = q:query('SELECT * FROM temperature WHERE time > now() - 24h')

-- 查询指定时间段
local result = q:query([[SELECT * FROM temperature WHERE time > '2021-01-01' AND time < '2021-01-02']])

-- 使用绝对时间
local result = q:query("SELECT * FROM temperature WHERE time > 1609459200000000000")
```

### 分组查询

```lua
-- 按时间分组（5分钟）
local result = q:query('SELECT MEAN(value) FROM temperature GROUP BY 5m')

-- 按标签分组
local result = q:query('SELECT MEAN(value) FROM temperature GROUP BY location')

-- 按时间和标签分组
local result = q:query('SELECT MEAN(value) FROM temperature GROUP BY 5m, location')

-- 填充零值
local result = q:query('SELECT COUNT(value) FROM temperature GROUP BY 5m fill(0)')
```

### 标签过滤

```lua
-- 等于
local result = q:query([[SELECT * FROM temperature WHERE location = 'room1']])

-- 不等于
local result = q:query([[SELECT * FROM temperature WHERE location != 'room1']])

-- 正则匹配
local result = q:query([[SELECT * FROM temperature WHERE location =~ /room.*/]])

-- 包含（IN）
local result = q:query([[SELECT * FROM temperature WHERE location IN ('room1', 'room2')]])
```

### 处理查询结果

```lua
local result = q:query('SELECT * FROM temperature WHERE time > now() - 1h')

if result and result.results then
    for _, res in ipairs(result.results) do
        if res.series then
            for _, series in ipairs(res.series) do
                print('Series:', series.name)

                -- 打印列名
                print('Columns:', table.concat(series.columns, ', '))

                -- 打印数据
                for _, row in ipairs(series.values) do
                    local time = os.date('%Y-%m-%d %H:%M:%S', row[1] / 1000000000)
                    local value = row[2]

                    print(string.format('  %s: %s', time, value))

                    -- 如果有标签
                    for i, col in ipairs(series.columns) do
                        if col ~= 'time' and col ~= 'value' then
                            print(string.format('    %s: %s', col, row[i]))
                        end
                    end
                end
            end
        end
    end
end
```

## InfluxQL参考

### SELECT子句

```lua
-- 选择所有字段
'SELECT * FROM measurement'

-- 选择指定字段
'SELECT field1, field2 FROM measurement'

-- 使用别名
'SELECT MEAN(field1) AS avg_field1 FROM measurement'

-- 多个聚合函数
'SELECT MEAN(value), MAX(value), MIN(value) FROM measurement'
```

### WHERE子句

```lua
-- 时间条件
'WHERE time > now() - 1h'
'WHERE time < now() - 24h'
'WHERE time >= "2021-01-01" AND time <= "2021-01-31"'

-- 字段条件
'WHERE value > 100'
"WHERE status = 'online'"
"WHERE value BETWEEN 0 AND 100"

-- 逻辑运算
'WHERE value > 50 AND location = "room1"'
'WHERE value < 0 OR value > 100'
'WHERE NOT (value < 0)'
```

### GROUP BY子句

```lua
-- 按时间分组
'GROUP BY 10m'     -- 10分钟
'GROUP BY 1h'      -- 1小时
'GROUP BY 1d'      -- 1天

-- 按标签分组
'GROUP BY location'
'GROUP BY device_type'

-- 多维度分组
'GROUP BY 5m, location'

-- 填充策略
'GROUP BY 1h fill(0)'          -- 填充0
'GROUP BY 1h fill(null)'       -- 填充null
'GROUP BY 1h fill(previous)'   -- 使用前一个值
'GROUP BY 1h fill(linear)'     -- 线性插值
```

### ORDER BY子句

```lua
-- 按时间排序（默认）
'ORDER BY time ASC'   -- 升序
'ORDER BY time DESC'  -- 降序

-- 按值排序
'ORDER BY value DESC'

-- 多字段排序
'ORDER BY time DESC, value ASC'
```

### LIMIT子句

```lua
-- 限制返回行数
'LIMIT 100'

-- 限制偏移量
'LIMIT 100 OFFSET 200'
```

## 常用查询模式

### 最新数据

```lua
-- 获取最新的一条数据
local result = q:query('SELECT * FROM temperature ORDER BY time DESC LIMIT 1')

-- 获取每个系列的最新数据
local result = q:query('SELECT * FROM temperature GROUP BY * ORDER BY time DESC LIMIT 1')
```

### 时间序列降采样

```lua
-- 将1秒采样降采样为1分钟
local result = q:query([[
    SELECT MEAN(value) INTO temperature_1m
    FROM temperature
    GROUP BY 1m
]])

-- 使用连续查询自动降采样
local result = q:query([[
    CREATE CONTINUOUS QUERY cq_temperature_1m
    ON sensors
    BEGIN
        SELECT MEAN(value) INTO temperature_1m
        FROM temperature
        GROUP BY 5m
    END
]])
```

### 数据对齐

```lua
-- 获取固定间隔的数据点
local result = q:query([[
    SELECT MEAN(value)
    FROM temperature
    WHERE time > now() - 1h
    GROUP BY 5m fill(0)
]])
```

### 系列发现

```lua
-- 查看所有measurement
local result = q:query('SHOW MEASUREMENTS')

-- 查看指定measurement的标签
local result = q:query('SHOW TAG KEYS FROM temperature')

-- 查看标签值
local result = q:query([[SHOW TAG VALUES FROM temperature WITH KEY = location]])

-- 查看字段
local result = q:query('SHOW FIELD KEYS FROM temperature')
```

## 错误处理

```lua
local result, err = q:query('SELECT * FROM temperature')

if not result then
    -- 处理错误
    if err:match('database not found') then
        print('Database does not exist')
    elseif err:match('authentication failed') then
        print('Invalid credentials')
    elseif err:match('syntax error') then
        print('Invalid query syntax')
    else
        print('Query failed:', err)
    end
end
```

## 性能优化

### 查询优化技巧

```lua
-- 1. 限制时间范围
'WHERE time > now() - 1h'  -- 好
-- （不要查询所有数据）

-- 2. 使用标签过滤
'WHERE location = "room1"'  -- 好
-- （标签索引比字段过滤快）

-- 3. 限制返回数量
'LIMIT 1000'

-- 4. 避免正则表达式
'WHERE device = "device1"'  -- 好
"WHERE device =~ /device.*/"  -- 差

-- 5. 使用连续查询预计算
-- 预先计算聚合结果
```

### 批量查询

```lua
-- 使用多个SELECT语句
local result = q:query([[
    SELECT MEAN(value) FROM temperature WHERE location = 'room1';
    SELECT MEAN(value) FROM temperature WHERE location = 'room2';
    SELECT MEAN(value) FROM temperature WHERE location = 'room3'
]])
```

## 完整应用示例

```lua
local query = require 'db.influxdb.query'

local function monitor_sensors()
    local q = query:new({
        host = 'influxdb.example.com',
        port = 8086,
        username = 'admin',
        password = 'password',
        database = 'sensors'
    })

    -- 查询所有传感器最新状态
    local result = q:query([[
        SELECT last(value)
        FROM temperature
        GROUP BY location
    ]])

    if result and result.results then
        for _, res in ipairs(result.results) do
            if res.series then
                for _, series in ipairs(res.series) do
                    local location = series.tags.location
                    local value = series.values[1][2]

                    if value > 30 then
                        print(string.format('WARNING: %s temperature: %.1f°C',
                            location, value))
                    else
                        print(string.format('%s: %.1f°C', location, value))
                    end
                end
            end
        end
    end
end

-- 定期监控
local function start_monitoring(interval)
    skynet.fork(function()
        while true do
            monitor_sensors()
            skynet.sleep(interval * 100)
        end
    end)
end

start_monitoring(60)  -- 每60秒检查一次
```
