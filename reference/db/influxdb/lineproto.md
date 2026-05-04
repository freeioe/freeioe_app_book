---

# InfluxDB行协议模块 (db.influxdb.lineproto)

本模块提供InfluxDB行协议（Line Protocol）的编码功能，用于构建符合InfluxDB写入格式的数据。

## 功能说明

InfluxDB行协议是InfluxDB的文本格式写入协议，格式如下：

```
measurement,tag_set field_set timestamp
```

例如：
```
temperature,location=room1,sensor=S001 value=25.5 1609459200000000000
```

## 使用方法

```lua
local lineproto = require 'db.influxdb.lineproto'

-- 引用字段值
local value = lineproto.quote_field_value(123)
print(value)  --> "123"

local value = lineproto.quote_field_value("string_value")
print(value)  --> "\"string_value\""

-- 构建标签集
local tags = {{method='GET'}, {status='200'}}
local tag_set = lineproto.build_tag_set(tags)

-- 构建字段集
local fields = {{value=100}, {count=1}}
local field_set = lineproto.build_field_set(fields)

-- 完整示例
local influx = {
    _measurement = lineproto.quote_measurement('temperature'),
    _tag_set = tag_set,
    _field_set = field_set,
    _stamp = 1609459200000000000
}

local line = lineproto.build_line_proto_stmt(influx)
```

## 接口说明

### quote_field_value

引用字段值（根据类型添加引号）

#### 函数原型

```lua
function lineproto.quote_field_value(value)
end
```

#### 参数说明

* value
  字段值（字符串）

#### 返回值

返回适当引用的字符串

#### 引用规则

- **数字**：不加引号
  - `123` → `123`
  - `123i` → `123i`（整数）
- **布尔值**：不加引号
  - `true`, `True`, `TRUE`, `t` → `true`
  - `false`, `False`, `FALSE`, `f` → `false`
- **字符串**：加双引号
  - 转义内部的双引号
  - `hello` → `"hello"`
  - `say "hi"` → `"say \"hi\""`

#### 示例

```lua
-- 数字
print(lineproto.quote_field_value('123'))      --> 123
print(lineproto.quote_field_value('123i'))     --> 123i
print(lineproto.quote_field_value('3.14'))     --> 3.14

-- 布尔值
print(lineproto.quote_field_value('true'))     --> true
print(lineproto.quote_field_value('FALSE'))    --> false

-- 字符串
print(lineproto.quote_field_value('hello'))    --> "hello"
print(lineproto.quote_field_value('say "hi"')) --> "say \"hi\""
```

### quote_field_key

引用字段键

#### 函数原型

```lua
function lineproto.quote_field_key(key)
end
```

#### 参数说明

* key
  字段键字符串

#### 返回值

返回转义后的字符串

#### 转义规则

- 逗号`,` → `\,`
- 等号`=` → `\=`
- 空格` ` → `\ `

#### 示例

```lua
print(lineproto.quote_field_key('field_name'))        --> field_name
print(lineproto.quote_field_key('field,name'))        --> field\,name
print(lineproto.quote_field_key('field=name'))        --> field\=name
print(lineproto.quote_field_key('field name'))        --> field\ name
```

### quote_tag_part

引用标签部分（键或值）

#### 函数原型

```lua
function lineproto.quote_tag_part(value)
end
```

#### 参数说明

* value
  标签键或值

#### 返回值

返回转义后的字符串

#### 转义规则

- 逗号`,` → `\,`
- 等号`=` → `\=`
- 空格` ` → `\ `

#### 示例

```lua
print(lineproto.quote_tag_part('location'))     --> location
print(lineproto.quote_tag_part('room,1'))       --> room\,1
print(lineproto.quote_tag_part('key=value'))    --> key\=value
print(lineproto.quote_tag_part('tag name'))     --> tag\ name
```

### quote_measurement

引用measurement名称

#### 函数原型

```lua
function lineproto.quote_measurement(name)
end
```

#### 参数说明

* name
  measurement名称

#### 返回值

返回转义后的字符串

#### 转义规则

- 逗号`,` → `\,`
- 空格` ` → `\ `

#### 示例

```lua
print(lineproto.quote_measurement('temperature'))      --> temperature
print(lineproto.quote_measurement('temp,sensor'))      --> temp\,sensor
print(lineproto.quote_measurement('my measurement'))   --> my\ measurement
```

### build_tag_set

构建标签集

#### 函数原型

```lua
function lineproto.build_tag_set(tags)
end
```

#### 参数说明

* tags
  标签数组，每个元素是单个键值对表

#### 返回值

返回编码后的标签集数组，格式为 `{"key1=value1", "key2=value2"}`

#### 示例

```lua
local tags = {
    {location = 'room1'},
    {sensor = 'S001'}
}

local tag_set = lineproto.build_tag_set(tags)
-- {'location=room1', 'sensor=S001'}
```

### build_field_set

构建字段集

#### 函数原型

```lua
function lineproto.build_field_set(fields)
end
```

#### 参数说明

* fields
  字段数组，每个元素是单个键值对表

#### 返回值

返回编码后的字段集数组，格式为 `{"key1=value1", "key2=value2"}`

#### 示例

```lua
local fields = {
    {value = 25.5},
    {status = 'OK'}
}

local field_set = lineproto.build_field_set(fields)
-- {'value=25.5', 'status="OK"'}
```

### build_line_proto_stmt

构建完整的行协议语句

#### 函数原型

```lua
function lineproto.build_line_proto_stmt(influx)
end
```

#### 参数说明

* influx
  InfluxDB对象表，包含以下字段：
  * `_measurement`：引用后的measurement名称
  * `_tag_set`：标签集数组
  * `_field_set`：字段集数组
  * `_stamp`：时间戳（纳秒）

#### 返回值

返回完整的行协议字符串

#### 格式

```
measurement,tag_set field_set timestamp
```

或没有标签时：

```
measurement field_set timestamp
```

## 完整示例

### 基本用法

```lua
local lineproto = require 'db.influxdb.lineproto'

-- 准备数据
local measurement = 'temperature'
local tags = {
    {location = 'room1'},
    {sensor_id = 'S001'}
}
local fields = {
    {value = 25.5},
    {unit = 'celsius'}
}
local timestamp = 1609459200000000000

-- 构建行协议
local influx = {
    _measurement = lineproto.quote_measurement(measurement),
    _tag_set = table.concat(lineproto.build_tag_set(tags), ','),
    _field_set = table.concat(lineproto.build_field_set(fields), ','),
    _stamp = timestamp
}

local line = lineproto.build_line_proto_stmt(influx)
print(line)
-- 输出: temperature,location=room1,sensor_id=S001 value=25.5,unit="celsius" 1609459200000000000
```

### 数据采集类

```lua
local lineproto = require 'db.influxdb.lineproto'

local influx_writer = {}
influx_writer.__index = influx_writer

function influx_writer:new(measurement)
    local obj = {
        _measurement = measurement,
        _tags = {},
        _fields = {}
    }
    return setmetatable(obj, self)
end

-- 添加标签
function influx_writer:add_tag(key, value)
    table.insert(self._tags, {[key] = value})
    return self
end

-- 添加字段
function influx_writer:add_field(key, value)
    table.insert(self._fields, {[key] = value})
    return self
end

-- 构建行协议
function influx_writer:build(timestamp)
    timestamp = timestamp or (skynet.time() * 1e9)

    local influx = {
        _measurement = lineproto.quote_measurement(self._measurement),
        _tag_set = table.concat(lineproto.build_tag_set(self._tags), ','),
        _field_set = table.concat(lineproto.build_field_set(self._fields), ','),
        _stamp = timestamp
    }

    return lineproto.build_line_proto_stmt(influx)
end

-- 使用示例
local writer = influx_writer:new('temperature')
writer:add_tag('location', 'room1')
writer:add_tag('sensor', 'S001')
writer:add_field('value', 25.5)
writer:add_field('quality', 'good')

local line = writer:build()
print(line)
```

### 批量写入

```lua
local lineproto = require 'db.influxdb.lineproto'

-- 收集多个数据点
local function collect_batch_data(readings)
    local lines = {}

    for _, reading in ipairs(readings) do
        local influx = {
            _measurement = lineproto.quote_measurement(reading.measurement),
            _tag_set = table.concat(lineproto.build_tag_set(reading.tags), ','),
            _field_set = table.concat(lineproto.build_field_set(reading.fields), ','),
            _stamp = reading.timestamp or (skynet.time() * 1e9)
        }

        table.insert(lines, lineproto.build_line_proto_stmt(influx))
    end

    return table.concat(lines, '\n')
end

-- 使用
local batch = {
    {
        measurement = 'temperature',
        tags = {{location='room1', sensor='S001'}},
        fields = {{value=25.5}},
        timestamp = 1609459200000000000
    },
    {
        measurement = 'humidity',
        tags = {{location='room1', sensor='S002'}},
        fields = {{value=65.2}},
        timestamp = 1609459201000000000
    }
}

local data = collect_batch_data(batch)
print(data)
```

## 最佳实践

1. **标签设计**：
   - 标签用于索引和分组
   - 避免高基数标签（如user_id）
   - 标签值应该是有限的枚举值

2. **字段类型**：
   - 字段值可以是数字、字符串或布尔值
   - 数字不带引号
   - 字符串必须加引号
   - 每个点至少有一个字段

3. **时间戳**：
   - 使用纳秒精度
   - 从毫秒转换：`timestamp_ms * 1e6`
   - 从秒转换：`timestamp_s * 1e9`

4. **性能优化**：
   - 批量写入使用`\n`分隔多行
   - 按measurement分组
   - 避免过多的标签组合

## 转义规则总结

| 字段 | 转义字符 |
| :--- | :--- |
| Measurement | `,` → `\,`, ` ` → `\ ` |
| Tag Key | `,` → `\,`, `=` → `\=`, ` ` → `\ ` |
| Tag Value | `,` → `\,`, `=` → `\=`, ` ` → `\ ` |
| Field Key | `,` → `\,`, `=` → `\=`, ` ` → `\ ` |
| Field Value (字符串) | `"` → `\"` |
| Field Value (数字/布尔) | 无需转义 |

## 错误处理

```lua
-- 检查measurement名称
local function safe_quote_measurement(name)
    if type(name) ~= 'string' then
        error('Measurement must be a string')
    end
    if name == '' then
        error('Measurement cannot be empty')
    end
    return lineproto.quote_measurement(name)
end

-- 检查字段集
local function validate_fields(fields)
    if not fields or #fields == 0 then
        error('At least one field is required')
    end
    return lineproto.build_field_set(fields)
end
```
