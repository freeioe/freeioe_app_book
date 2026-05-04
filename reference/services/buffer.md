

---

# 缓存服务

FreeIOE 提供的数据缓存服务。管理系统通信数据、日志和事件的缓存。

## 获取服务实例

```lua
local buffer = snax.queryservice('buffer')
```

## 响应接口 (req/response)

### ping

健康检查

#### 函数原型

```lua
function buffer.req.ping()
end
```

#### 返回值

返回 "PONG"

### get_comm

获取通信数据缓存

#### 函数原型

```lua
function buffer.req.get_comm(app)
end
```

#### 参数说明

* app
  应用名称（可选，nil返回所有应用的通信数据）

#### 返回值

返回通信数据缓存表

### get_log

获取日志缓存

#### 函数原型

```lua
function buffer.req.get_log(app)
end
```

#### 参数说明

* app
  应用名称（可选，nil返回所有日志）

#### 返回值

返回日志缓存表

### get_event

获取事件缓存

#### 函数原型

```lua
function buffer.req.get_event()
end
```

#### 返回值

返回事件缓存表

## 接口调用 (accept/post)

### listen

注册缓存事件监听器

#### 函数原型

```lua
function buffer.post.listen(handle, handle_type)
end
```

#### 参数说明

* handle
  监听器服务句柄
* handle_type
  监听器服务类型

### unlisten

取消缓存事件监听器

#### 函数原型

```lua
function buffer.post.unlisten(handle, handle_type)
end
```

#### 参数说明

* handle
  监听器服务句柄
* handle_type
  监听器服务类型

## 缓存数据格式

### 通信数据

```lua
{
    sn = "device_serial_number",
    dir = "send" or "recv",
    ts = timestamp,
    data = "base64_encoded_data"
}
```

### 日志数据

```lua
{
    timestamp = timestamp,
    level = "INFO" or "DEBUG" etc,
    process = "process_name",
    content = "log message"
}
```

### 事件数据

```lua
{
    app = "app_name",
    sn = "device_serial_number",
    level = event_level,
    type = event_type,
    info = "event description",
    data = {},
    timestamp = timestamp
}
```
