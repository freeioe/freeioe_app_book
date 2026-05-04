

---

# 应用管理服务

FreeIOE 提供的应用管理服务。

## 获取服务实例

```lua
local appmgr = snax.queryservice('appmgr')
```

## 响应接口 (req/response)

### start

启动应用

#### 函数原型

```lua
function appmgr.req.start(name, conf, skip_mode_check)
end
```

#### 参数说明

* name
  应用实例名称
* conf
  应用配置（可选）
* skip_mode_check
  是否跳过模式检查（可选，默认false）

#### 返回值

成功返回应用实例，失败返回nil和错误消息

#### 示例代码

```lua
local appmgr = snax.queryservice('appmgr')
local r, err = appmgr.req.start('modbus_1', conf)
```

### stop

停止应用

#### 函数原型

```lua
function appmgr.req.stop(name, reason)
end
```

#### 参数说明

* name
  应用实例名称
* reason
  停止原因（可选）

#### 返回值

成功返回true，失败返回nil和错误消息

### restart

重启应用

#### 函数原型

```lua
function appmgr.req.restart(name, reason)
end
```

#### 参数说明

* name
  应用实例名称
* reason
  重启原因（可选）

#### 返回值

成功返回应用实例，失败返回false和错误消息

#### 示例代码

```lua
local appmgr = snax.queryservice('appmgr')
local r, err = appmgr.req.restart('modbus_1', 'restart app from user')
```

### list

获取应用列表

#### 函数原型

```lua
function appmgr.req.list()
end
```

#### 返回值

返回应用列表表，包含每个应用的信息

#### 示例代码

```lua
local appmgr = snax.queryservice('appmgr')
local apps = appmgr.req.list()
```

### app_inst

获取应用实例

#### 函数原型

```lua
function appmgr.req.app_inst(name)
end
```

#### 参数说明

* name
  应用实例名称

#### 返回值

返回应用实例句柄，如果应用不存在或未启动则返回nil

### get_conf

获取应用配置

#### 函数原型

```lua
function appmgr.req.get_conf(inst)
end
```

#### 参数说明

* inst
  应用实例名称

#### 返回值

返回应用配置表

#### 示例代码

```lua
local appmgr = snax.queryservice('appmgr')
local conf, err = appmgr.req.get_conf('modbus_1')
```

### set_conf

更改应用配置

#### 函数原型

```lua
function appmgr.req.set_conf(inst, conf)
end
```

#### 参数说明

* inst
  应用实例名称
* conf
  新的配置表

#### 返回值

成功返回true，失败返回nil和错误消息

#### 示例代码

```lua
local appmgr = snax.queryservice('appmgr')
local r, err = appmgr.req.set_conf('modbus_1', conf)
```

### app_option

更改应用选项

#### 函数原型

```lua
function appmgr.req.app_option(inst, option, value)
end
```

#### 参数说明

* inst
  应用实例名称
* option
  选项名称（如'auto'用于自动启动）
* value
  选项值

#### 返回值

成功返回true，失败返回nil和错误消息

#### 示例代码

```lua
local appmgr = snax.queryservice('appmgr')
local r, err = appmgr.req.app_option('modbus_1', 'auto', 1)
```

### app_rename

应用改名

#### 函数原型

```lua
function appmgr.req.app_rename(inst, new_name, reason)
end
```

#### 参数说明

* inst
  原应用实例名称
* new_name
  新的应用实例名称
* reason
  改名原因（可选）

#### 返回值

成功返回true，失败返回nil和错误消息

#### 示例代码

```lua
local appmgr = snax.queryservice('appmgr')
local r, err = appmgr.req.app_rename('modbus_1', 'modbus_2', "change the name by cloud")
```

### get_channel

获取多播通道

#### 函数原型

```lua
function appmgr.req.get_channel(name)
end
```

#### 参数说明

* name
  通道名称（DATA, CTRL, COMM, STAT, EVENT）

#### 返回值

返回多播通道对象，如果不存在则返回nil

## 接口调用 (accept/post)

### app_start

启动应用（异步）

```lua
appmgr.post.app_start(inst, skip_mode_check)
```

### app_stop

停止应用（异步）

```lua
appmgr.post.app_stop(inst, reason)
```

### app_restart

重启应用（异步）

```lua
appmgr.post.app_restart(inst, reason)
```

### dev_app_start

启动开发应用（异步）

```lua
appmgr.post.dev_app_start(app_path)
```

### app_event

触发应用事件

```lua
appmgr.post.app_event(event, inst_name, ...)
```

### app_heartbeat

应用心跳

```lua
appmgr.post.app_heartbeat(inst, time)
```

### fire_event

触发事件到事件通道

```lua
appmgr.post.fire_event(app_name, sn, level, type_, info, data, timestamp)
```

### listen

监听应用事件

```lua
appmgr.post.listen(handle, type, fire_list)
```

### unlisten

取消监听

```lua
appmgr.post.unlisten(handle)
```

### close_all

停止所有应用

#### 函数原型

```lua
appmgr.post.close_all(reason)
```

#### 参数说明

* reason
  停止所有应用的原因

#### 示例代码

```lua
local appmgr = snax.queryservice('appmgr')
appmgr.post.close_all("close all applications by cloud")
```
