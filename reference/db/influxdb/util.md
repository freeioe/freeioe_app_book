---

# InfluxDB工具模块 (db.influxdb.util)

本模块提供InfluxDB的底层工具函数，包括HTTP/UDP写入、查询和配置验证功能。

## 功能说明

工具模块提供了与InfluxDB交互的基础功能：

- **HTTP写入**：通过HTTP API写入数据
- **UDP写入**：通过UDP协议写入数据
- **HTTP查询**：执行InfluxQL查询
- **选项验证**：验证和规范化配置选项

此模块通常被其他高级模块（如buffer、object）内部使用，也可以直接使用。

## 接口说明

### write_http

通过HTTP写入数据

#### 函数原型

```lua
function util.write_http(msg, params)
end
```

#### 参数说明

* msg
  要写入的消息字符串（InfluxDB行协议格式）
* params
  参数表

| 选项 | 类型 | 默认值 | 说明 |
| :--- | :--- | :--- | :--- |
| host | string | '127.0.0.1' | InfluxDB主机地址 |
| port | number | 8086 | InfluxDB端口 |
| db | string | 'influx' | 数据库名称 |
| precision | string | 'ms' | 时间精度 |
| ssl | boolean | false | 是否使用SSL/TLS |
| auth | string | nil | 认证信息（username:password格式） |

#### 返回值

成功返回true和响应体，失败返回false和状态码

#### HTTP状态码

- **204**：写入成功（No Content）
- 其他：写入失败

#### 示例

```lua
local util = require 'db.influxdb.util'

-- 准备行协议数据
local msg = 'temperature,location=room1 value=25.5 1609459200000000000'

-- 写入数据
local ok, body = util.write_http(msg, {
    host = 'localhost',
    port = 8086,
    db = 'sensors',
    precision = 'ms'
})

if ok then
    print('Write success')
else
    print('Write failed:', body)
end
```

### write_udp

通过UDP写入数据

#### 函数原型

```lua
function util.write_udp(msg, params)
end
```

#### 参数说明

* msg
  要写入的消息字符串
* params
  参数表

| 选项 | 类型 | 默认值 | 说明 |
| :--- | :--- | :--- | :--- |
| host | string | '127.0.0.1' | InfluxDB主机地址 |
| port | number | 8089 | UDP端口 |

#### 返回值

成功返回true

#### 注意事项

> UDP写入不保证送达，没有响应确认。

#### 示例

```lua
local util = require 'db.influxdb.util'

-- 准备数据
local msg = 'temperature,location=room1 value=25.5'

-- UDP写入
util.write_udp(msg, {
    host = 'localhost',
    port = 8089
})

print('UDP write sent')
```

### query_http

通过HTTP执行查询

#### 函数原型

```lua
function util.query_http(params)
end
```

#### 参数说明

* params
  参数表

| 选项 | 类型 | 默认值 | 说明 |
| :--- | :--- | :--- | :--- |
| host | string | '127.0.0.1' | InfluxDB主机地址 |
| port | number | 8086 | InfluxDB端口 |
| db | string | 必需 | 数据库名称 |
| precision | string | 'ms' | 时间精度 |
| ssl | boolean | false | 是否使用SSL/TLS |
| auth | string | nil | 认证信息 |
| username | string | nil | 用户名（如果auth未设置） |
| password | string | nil | 密码（如果auth未设置） |
| query | string | 必需 | InfluxQL查询语句 |

#### 返回值

成功返回true和响应体，失败返回false和状态码

#### 示例

```lua
local util = require 'db.influxdb.util'

-- 执行查询
local ok, body = util.query_http({
    host = 'localhost',
    port = 8086,
    db = 'sensors',
    query = 'SELECT * FROM temperature WHERE time > now() - 1h'
})

if ok then
    local cjson = require 'cjson'
    local result = cjson.decode(body)
    print('Query result:', result)
else
    print('Query failed:', body)
end
```

### validate_options

验证配置选项

#### 函数原型

```lua
function util.validate_options(opts)
end
```

#### 参数说明

* opts
  配置选项表

#### 返回值

验证通过返回true，失败返回false和错误信息

#### 验证规则

| 参数 | 验证规则 |
| :--- | :--- |
| opts | 必须是table类型 |
| host | 必须是string类型 |
| port | 必须是number类型，范围0-65535 |
| db | 必须是string类型，不能为空 |
| hostname | 必须是string类型 |
| proto | 必须是'http'或'udp' |
| precision | 必须是string类型 |
| ssl | 必须是boolean类型 |
| auth | 如果存在，必须是string类型 |

#### 默认值设置

此函数会为未设置的选项设置默认值：

```lua
opts.host      = opts.host or '127.0.0.1'
opts.port      = opts.port or 8086
opts.db        = opts.db or 'influx'
opts.hostname  = opts.hostname or opts.host
opts.proto     = opts.proto or 'http'
opts.precision = opts.precision or 'ms'
opts.ssl       = opts.ssl or false
```

#### 示例

```lua
local util = require 'db.influxdb.util'

-- 验证配置
local ok, err = util.validate_options({
    host = 'localhost',
    port = 8086,
    db = 'mydb',
    proto = 'http',
    precision = 'ms'
})

if not ok then
    print('Invalid options:', err)
    return
end

print('Options validated')
```

### init_udp

初始化UDP连接

#### 函数原型

```lua
function util.init_udp()
end
```

#### 说明

此函数初始化UDP socket，通常在第一次使用UDP写入时自动调用。

#### 示例

```lua
local util = require 'db.influxdb.util'

-- 手动初始化UDP
util.init_udp()
```

## 完整示例

### 基本HTTP写入

```lua
local util = require 'db.influxdb.util'

-- 构建行协议数据
local function build_line_proto(measurement, tags, fields, timestamp)
    local line = measurement

    -- 添加标签
    if tags then
        local tag_parts = {}
        for k, v in pairs(tags) do
            table.insert(tag_parts, k .. '=' .. v)
        end
        if #tag_parts > 0 then
            line = line .. ',' .. table.concat(tag_parts, ',')
        end
    end

    -- 添加字段
    local field_parts = {}
    for k, v in pairs(fields) do
        table.insert(field_parts, k .. '=' .. tostring(v))
    end
    line = line .. ' ' .. table.concat(field_parts, ',')

    -- 添加时间戳
    if timestamp then
        line = line .. ' ' .. timestamp
    end

    return line
end

-- 写入数据
local line = build_line_proto(
    'temperature',
    {location = 'room1', sensor = 'S001'},
    {value = 25.5},
    os.time() * 1000
)

local ok, body = util.write_http(line, {
    host = 'localhost',
    port = 8086,
    db = 'sensors',
    precision = 'ms'
})

print('Write result:', ok, body)
```

### 批量HTTP写入

```lua
local util = require 'db.influxdb.util'

-- 批量写入
local function batch_write(lines, params)
    -- 合并多行，用换行符分隔
    local msg = table.concat(lines, '\n')

    return util.write_http(msg, params)
end

-- 准备批量数据
local lines = {}
for i = 1, 100 do
    table.insert(lines, string.format(
        'metric,id=%d value=%f %d',
        i,
        math.random() * 100,
        os.time() * 1000
    ))
end

-- 批量写入
local ok, body = batch_write(lines, {
    host = 'localhost',
    port = 8086,
    db = 'metrics',
    precision = 'ms'
})

print('Batch write result:', ok)
```

### UDP高频写入

```lua
local util = require 'db.influxdb.util'

-- 初始化UDP
util.init_udp()

-- 高频数据写入
local start_time = os.time()

for i = 1, 1000 do
    local line = string.format(
        'vibration,sensor=%d x=%f,y=%f,z=%f',
        i,
        math.random(),
        math.random(),
        math.random()
    )

    util.write_udp(line, {
        host = 'localhost',
        port = 8089
    })
end

local elapsed = os.time() - start_time
print(string.format('Sent 1000 UDP packets in %d seconds', elapsed))
```

### 查询数据处理

```lua
local util = require 'db.influxdb.util'
local cjson = require 'cjson'

-- 查询并处理结果
local function query_and_process(sql)
    local ok, body = util.query_http({
        host = 'localhost',
        port = 8086,
        db = 'sensors',
        query = sql
    })

    if not ok then
        print('Query failed:', body)
        return nil
    end

    -- 解析JSON响应
    local result = cjson.decode(body)

    -- 检查结果
    if result and result.results then
        for _, res in ipairs(result.results) do
            if res.series then
                for _, series in ipairs(res.series) do
                    print('Series:', series.name)
                    print('Columns:', table.concat(series.columns, ', '))

                    for _, row in ipairs(series.values) do
                        print('  ', table.concat(row, ', '))
                    end
                end
            end
        end
    end

    return result
end

-- 使用示例
query_and_process('SELECT * FROM temperature LIMIT 10')
```

### 配置验证

```lua
local util = require 'db.influxdb.util'

-- 定义配置模板
local configs = {
    -- 正确配置
    valid = {
        host = 'localhost',
        port = 8086,
        db = 'production',
        proto = 'http',
        precision = 'ms'
    },

    -- 错误的端口
    invalid_port = {
        host = 'localhost',
        port = 99999,  -- 超出范围
        db = 'test'
    },

    -- 错误的协议
    invalid_proto = {
        host = 'localhost',
        port = 8086,
        db = 'test',
        proto = 'tcp'  -- 不是http或udp
    },

    -- 空数据库名
    invalid_db = {
        host = 'localhost',
        port = 8086,
        db = ''  -- 空字符串
    }
}

-- 验证配置
for name, config in pairs(configs) do
    local ok, err = util.validate_options(config)
    print(name, ':', ok and 'VALID' or 'INVALID', err or '')
end
```

### 带认证的操作

```lua
local util = require 'db.influxdb.util'

-- 认证信息格式：username:password
local auth = 'admin:secret_password'

-- 带认证的写入
local ok, body = util.write_http(msg, {
    host = 'secure-influxdb.example.com',
    port = 8086,
    db = 'production',
    ssl = true,
    auth = auth
})

-- 带认证的查询
local qok, qbody = util.query_http({
    host = 'secure-influxdb.example.com',
    port = 8086,
    db = 'production',
    ssl = true,
    auth = auth,
    query = 'SHOW DATABASES'
})
```

### 错误处理

```lua
local util = require 'db.influxdb.util'

-- 安全写入函数
local function safe_write(msg, params)
    -- 先验证参数
    local ok, err = util.validate_options(params)
    if not ok then
        return nil, 'Invalid params: ' .. err
    end

    -- 执行写入
    local wok, body = util.write_http(msg, params)

    if not wok then
        -- 分析错误
        local status = tonumber(body)
        if status == 401 then
            return nil, 'Authentication failed'
        elseif status == 404 then
            return nil, 'Database not found'
        else
            return nil, 'Write failed with status: ' .. body
        end
    end

    return true
end

-- 使用
local result, err = safe_write(msg, params)
if not result then
    print('Error:', err)
end
```

### 性能测试

```lua
local util = require 'db.influxdb.util'

-- HTTP写入性能
local function benchmark_http(count)
    local start = os.clock()

    for i = 1, count do
        local msg = string.format('bench value=%d %d', i, os.time() * 1000)
        util.write_http(msg, {
            host = 'localhost',
            port = 8086,
            db = 'benchmark'
        })
    end

    local elapsed = os.clock() - start
    print(string.format('HTTP: %d writes in %.2f seconds (%.2f writes/sec)',
        count, elapsed, count / elapsed))
end

-- UDP写入性能
local function benchmark_udp(count)
    util.init_udp()
    local start = os.clock()

    for i = 1, count do
        local msg = string.format('bench value=%d', i)
        util.write_udp(msg, {
            host = 'localhost',
            port = 8089
        })
    end

    local elapsed = os.clock() - start
    print(string.format('UDP: %d writes in %.2f seconds (%.2f writes/sec)',
        count, elapsed, count / elapsed))
end

-- 运行测试
benchmark_http(100)
benchmark_udp(100)
```

## 最佳实践

1. **批量写入**：合并多个数据点用换行符分隔
2. **参数验证**：写入前先验证参数
3. **错误处理**：检查返回值并处理错误
4. **UDP初始化**：UDP使用前调用init_udp()
5. **认证安全**：使用auth参数传递认证信息
6. **时间精度**：确保时间戳与precision参数匹配

## 注意事项

1. **HTTP 204**：成功的HTTP写入返回204状态码
2. **UDP无确认**：UDP写入不保证送达
3. **查询转义**：查询字符串需要正确转义特殊字符
4. **认证格式**：auth字段格式为"username:password"
5. **默认值**：validate_options会设置默认值

## 与其他模块配合

```lua
-- 与lineproto模块配合
local lineproto = require 'db.influxdb.lineproto'
local util = require 'db.influxdb.util'

local influx_data = {
    _measurement = 'temperature',
    _tag_set = {{location='room1'}},
    _field_set = {{value=25.5}},
    _stamp = os.time() * 1000
}

local line = lineproto.build_line_proto_stmt(influx_data)
util.write_http(line, params)
```

## 错误排查

### 常见错误及解决方法

```lua
-- 1. 认证失败 (401)
-- 解决：检查auth字段格式是否为"username:password"

-- 2. 数据库不存在 (404)
-- 解决：先创建数据库或检查db参数

-- 3. 无效的行协议 (400)
-- 解决：检查行协议格式是否正确

-- 4. 连接超时
-- 解决：检查host和port是否正确

-- 5. 端口范围错误
-- 解决：port必须在0-65535范围内

-- 6. 协议错误
-- 解决：proto必须是'http'或'udp'
```
