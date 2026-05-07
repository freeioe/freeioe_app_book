---

# InfluxDB缓冲写入模块 (db.influxdb.buffer)

本模块提供InfluxDB的缓冲写入功能，支持批量写入以提高性能。

## 功能说明

缓冲写入模块允许将多个数据点先收集到内存缓冲区，然后一次性批量写入InfluxDB，从而减少网络开销和提高写入效率。

支持的写入协议：
- **HTTP**：通过HTTP API写入
- **UDP**：通过UDP协议写入（更适合高频写入场景）

## 使用方法

```lua
local buffer = require 'db.influxdb.buffer'

-- 创建缓冲区对象
local buf = buffer:new({
    proto = 'http',       -- 使用HTTP协议
    host = 'localhost',
    port = 8086,
    db = 'mydb',
    precision = 'ms',
    ssl = false
})

-- 添加数据到缓冲区
buf:buffer({
    measurement = 'temperature',
    tags = {location = 'room1', sensor = 'S001'},
    fields = {value = 25.5}
})

-- 添加更多数据
buf:buffer({
    measurement = 'humidity',
    tags = {location = 'room1'},
    fields = {value = 65.2}
})

-- 刷新缓冲区，写入所有数据
buf:flush()
```

## 接口说明

### new

创建新的缓冲区对象

#### 函数原型

```lua
function buffer:new(opts)
end
```

#### 参数说明

* opts
  配置选项表

| 选项 | 类型 | 默认值 | 说明 |
| :--- | :--- | :--- | :--- |
| proto | string | 'http' | 写入协议（http/udp） |
| host | string | '127.0.0.1' | InfluxDB主机地址 |
| port | number | 8086 | InfluxDB端口 |
| db | string | 'influx' | 数据库名称 |
| hostname | string | 同host | 主机名 |
| precision | string | 'ms' | 时间精度（s/ms/us/ns） |
| ssl | boolean | false | 是否使用SSL/TLS |
| auth | string | nil | 认证信息（username:password格式） |

#### 返回值

返回缓冲区对象

#### 示例

```lua
-- HTTP缓冲区
local http_buf = buffer:new({
    proto = 'http',
    host = 'localhost',
    port = 8086,
    db = 'sensors',
    precision = 'ms'
})

-- UDP缓冲区
local udp_buf = buffer:new({
    proto = 'udp',
    host = 'localhost',
    port = 8089,
    db = 'sensors',
    precision = 'ms'
})

-- 带认证的缓冲区
local auth_buf = buffer:new({
    proto = 'http',
    host = 'influxdb.example.com',
    port = 8086,
    db = 'production',
    auth = 'admin:secret_password'
})
```

### buffer

将数据添加到缓冲区

#### 函数原型

```lua
function buf:buffer(data)
end
```

#### 参数说明

* data
  数据表，包含以下字段：

| 字段 | 类型 | 说明 |
| :--- | :--- | :--- |
| measurement | string | 测量名称 |
| tags | table | 标签表（可选） |
| fields | table | 字段表（必须） |

#### 返回值

成功返回true

#### 示例

```lua
-- 添加带标签的数据
buf:buffer({
    measurement = 'temperature',
    tags = {
        location = 'room1',
        sensor = 'S001'
    },
    fields = {
        value = 25.5
    }
})

-- 添加不带标签的数据
buf:buffer({
    measurement = 'system_status',
    fields = {
        status = 'running',
        uptime = 123456
    }
})

-- 添加多个字段
buf:buffer({
    measurement = 'sensor_reading',
    tags = {id = 'SENSOR_01'},
    fields = {
        temperature = 25.5,
        humidity = 65.2,
        pressure = 1013.25
    }
})
```

### flush

刷新缓冲区，将所有缓冲的数据写入InfluxDB

#### 函数原型

```lua
function buf:flush()
end
```

#### 说明

此方法会：
1. 将缓冲区中的所有数据点合并为一条消息
2. 通过配置的协议（HTTP/UDP）发送
3. 清空缓冲区
4. 使用异步方式执行写入（通过skynet.fork）

#### 示例

```lua
-- 收集一批数据后刷新
for i = 1, 100 do
    buf:buffer({
        measurement = 'metric',
        tags = {id = tostring(i)},
        fields = {value = math.random()}
    })
end

-- 一次性写入所有数据
buf:flush()
```

### clear

清空缓冲区而不写入

#### 函数原型

```lua
function buf:clear()
end
```

#### 使用场景

- 丢弃不需要的数据
- 重置缓冲区状态
- 错误恢复

#### 示例

```lua
-- 添加数据
buf:buffer({...})

-- 遇到错误，清空缓冲区
if error_occurred then
    buf:clear()
end
```

## 完整示例

### 批量传感器数据收集

```lua
local buffer = require 'db.influxdb.buffer'
local skynet = require 'skynet'

-- 创建缓冲区
local buf = buffer:new({
    proto = 'http',
    host = 'localhost',
    port = 8086,
    db = 'sensors',
    precision = 'ms'
})

-- 模拟传感器读取
local sensors = {
    {id='S001', type='temperature', value=25.5},
    {id='S002', type='temperature', value=24.8},
    {id='S003', type='temperature', value=26.1},
    {id='H001', type='humidity', value=65.2},
    {id='H002', type='humidity', value=63.8},
}

-- 收集所有传感器数据
for _, sensor in ipairs(sensors) do
    buf:buffer({
        measurement = sensor.type,
        tags = {
            sensor_id = sensor.id,
            location = 'room1'
        },
        fields = {
            value = sensor.value
        }
    })
end

-- 批量写入
print('Flushing', #sensors, 'data points...')
buf:flush()
print('Flush complete')
```

### 定时批量写入

```lua
local skynet = require 'skynet'
local buffer = require 'db.influxdb.buffer'

local buf = buffer:new({
    proto = 'http',
    host = 'localhost',
    port = 8086,
    db = 'metrics',
    precision = 'ms'
})

-- 数据收集计数
local count = 0
local BATCH_SIZE = 100

-- 添加指标数据
local function add_metric(name, value, tags)
    buf:buffer({
        measurement = name,
        tags = tags or {},
        fields = {value = value}
    })

    count = count + 1

    -- 达到批量大小时刷新
    if count >= BATCH_SIZE then
        buf:flush()
        count = 0
    end
end

-- 使用示例
for i = 1, 500 do
    add_metric('cpu_usage', math.random() * 100, {cpu='cpu0'})
    skynet.sleep(10)  -- 等待100ms
end

-- 最后刷新剩余数据
buf:flush()
```

### UDP高频写入

```lua
local buffer = require 'db.influxdb.buffer'

-- UDP更适合高频写入
local udp_buf = buffer:new({
    proto = 'udp',
    host = 'localhost',
    port = 8089,  -- InfluxDB UDP端口
    db = 'high_frequency',
    precision = 'ms'
})

-- 高频数据采集（每秒1000次）
local start_time = skynet.time()
for i = 1, 1000 do
    udp_buf:buffer({
        measurement = 'vibration',
        tags = {sensor = 'accelerometer_1'},
        fields = {
            x = math.random(),
            y = math.random(),
            z = math.random()
        }
    })
    skynet.sleep(1)  -- 1ms间隔
end

local elapsed = skynet.time() - start_time
print(string.format('Collected 1000 samples in %.2f seconds', elapsed))

udp_buf:flush()
```

### 错误处理

```lua
local buffer = require 'db.influxdb.buffer'

local buf = buffer:new({
    proto = 'http',
    host = 'localhost',
    port = 8086,
    db = 'test',
    precision = 'ms'
})

-- 安全添加数据
local function safe_buffer(data)
    local ok, err = pcall(function()
        buf:buffer(data)
    end)

    if not ok then
        log.error('Buffer error:', err)
        return false
    end

    return true
end

-- 安全刷新
local function safe_flush()
    local ok, err = pcall(function()
        buf:flush()
    end)

    if not ok then
        log.error('Flush error:', err)
        -- 清空缓冲区避免重复写入
        buf:clear()
        return false
    end

    return true
end

-- 使用
safe_buffer({...})
safe_buffer({...})
safe_flush()
```

### 多缓冲区管理

```lua
local buffer = require 'db.influxdb.buffer'

-- 为不同的数据类型创建不同的缓冲区
local buffers = {
    metrics = buffer:new({
        proto = 'http',
        host = 'localhost',
        port = 8086,
        db = 'metrics'
    }),
    events = buffer:new({
        proto = 'http',
        host = 'localhost',
        port = 8086,
        db = 'events'
    }),
    debug = buffer:new({
        proto = 'udp',
        host = 'localhost',
        port = 8089,
        db = 'debug'
    })
}

-- 写入到不同的缓冲区
buffers.metrics:buffer({
    measurement = 'cpu_usage',
    fields = {value = 45.2}
})

buffers.events:buffer({
    measurement = 'app_start',
    tags = {app = 'myapp'},
    fields = {pid = 12345}
})

buffers.debug:buffer({
    measurement = 'debug_log',
    fields = {message = 'Debug info'}
})

-- 分别刷新
for name, buf in pairs(buffers) do
    print('Flushing', name)
    buf:flush()
end
```

## HTTP vs UDP

### HTTP写入

**优势：**
- 可靠传输，有响应确认
- 支持认证
- 支持SSL/TLS加密
- 标准InfluxDB接口

**劣势：**
- 开销较大
- 不适合极高频率写入

**适用场景：**
- 关键数据
- 需要确认的写入
- 网络不稳定环境

```lua
local http_buf = buffer:new({
    proto = 'http',
    host = 'localhost',
    port = 8086,
    db = 'production',
    ssl = true,
    auth = 'admin:password'
})
```

### UDP写入

**优势：**
- 低延迟
- 高吞吐量
- 适合高频写入

**劣势：**
- 不保证送达
- 无响应确认
- 不支持认证

**适用场景：**
- 高频数据采集
- 允许丢失部分数据
- 本地网络环境

```lua
local udp_buf = buffer:new({
    proto = 'udp',
    host = 'localhost',
    port = 8089,
    db = 'telemetry'
})
```

## 性能优化

### 批量大小

```lua
-- 找到最佳批量大小
local BATCH_SIZES = {10, 50, 100, 500, 1000}
local current_batch = 0
local batch_size = 100  -- 可调整

local function add_data(data)
    buf:buffer(data)
    current_batch = current_batch + 1

    if current_batch >= batch_size then
        buf:flush()
        current_batch = 0
    end
end
```

### 定时刷新

```lua
-- 定时刷新确保数据及时写入
local function start_periodic_flush(interval)
    skynet.fork(function()
        while true do
            skynet.sleep(interval * 100)
            buf:flush()
        end
    end)
end

-- 每5秒刷新一次
start_periodic_flush(5)
```

### 内存管理

```lua
-- 监控缓冲区大小
local function get_buffer_size()
    -- 估算缓冲区数据量
    return current_batch
end

-- 达到阈值时立即刷新
if get_buffer_size() > MAX_BUFFER_SIZE then
    buf:flush()
end
```

## 最佳实践

1. **批量大小**：根据数据量选择合适的批量大小（100-1000）
2. **定时刷新**：结合定时刷新避免数据丢失
3. **错误处理**：捕获写入错误并处理
4. **协议选择**：根据场景选择HTTP或UDP
5. **资源清理**：程序退出前调用flush()
6. **监控**：监控缓冲区大小和刷新频率

## 注意事项

1. **异步写入**：flush()是异步的，不会阻塞当前流程
2. **数据丢失**：进程异常退出时缓冲区数据会丢失
3. **内存使用**：大量缓冲会占用内存，需要控制大小
4. **网络依赖**：UDP写入不保证送达
5. **时间戳**：自动生成时间戳，使用系统时间

## 错误排查

```lua
-- 常见问题

-- 1. 连接失败
-- 检查host和port是否正确

-- 2. 认证失败
-- 验证auth字段格式：username:password

-- 3. 数据库不存在
-- 先创建数据库

-- 4. 批量过大
-- 减小批量大小，分批写入

-- 5. UDP端口错误
-- InfluxDB UDP端口默认8089，需在配置文件中启用
```
