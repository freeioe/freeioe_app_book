
----

# 应用基础接口

FreeIOE框架为每个应用创建的服务接口，用以帮助应用快速构建设备模型等操作

## set_handler

设定处理回调函数。

### 函数原型

```lua
function api:set_handler(handler, watch_data)
end
```

### 参数说明

* handler
  回调函数集合(table)
* watch_data
  是否关注其他应用创建的设备数据消息

### 示例代码

```lua
local api = sys:data_api()
api:set_handler({
	on_comm = function(src_app, dev_sn, dir, timestamp, ...) end, -- watch_data = true
	on_stat = function(src_app, dev_sn, state, prop, value, timestamp) end, -- watch_data = true
	on_input = function(src_app, dev_sn, input, prop, value, timestamp, quality) end, -- watch_data = true
	on_add_device = function(src_app, dev_sn, props) end, -- watch_data = true
	on_del_device = function(src_app, dev_sn, props) end, -- watch_data = true
	on_mod_device = function(src_app, dev_sn) end, -- watch_data = true
	on_output = function(src_app, dev_sn, output, prop, value, timestamp) end, -- 数据输出项回调
	on_output_result = function(src_app, priv, result, err) end, -- 数据输出项请求执行结果回调
	on_command = function(src_app, dev_sn, command, params) end, -- 命令回调
	on_command_result = function(src_app, priv, result, err) end, -- 命令请求执行结果回调
	on_ctrl = function(src_app, command, params) end, -- 应用控制接口
	on_ctrl_result = function(src_app, priv, result, err) end, -- 应用控制执行结果回调
})
```

> *** API_VER: 5 ***
> API_VER 5支持了 on_output_result 和 on_command_result接口

## list_devices

枚举系统中所有设备对象的描述信息 (meta, inputs, outputs, commands等等)

### 函数原型

```lua
function api:list_devices()
end
```

## add_device

创建新的采集设备对象。返回设备对象实例（参考[设备API](device.md))。

### 函数原型
```lua
function api:add_device(sn, meta, inputs, outputs, commands)
end
```

### 参数说明

* sn
  设备序列号
* meta
  设备Meta信息
* inputs
  设备输入项列表
* outputs
  设备输出项列表
* commands
  设备控制项列表

> 了解[更多信息](../../guide/app_dev/easy/data_collection.md)

### 示例代码

```lua
--- 创建设备模型
local inputs = {
	{name="tag1", desc="tag1 desc", unit="KV", vt="float"},
	{name="tag2", desc="tag2 desc", unit="KV", vt="float"}
}
local outputs = {
	{name="output1", desc="output1 desc", unit="V"},
}
local commands = {
	{name="command1", desc="command desc"},
}

local meta = self._api:default_meta()
meta.name = "Example Device"
meta.description = "Example Device Meta"
--- 生成设备模型对象
local dev = self._api:add_device(sn, meta, inputs, outputs, commands)
```

## del_device

删除设备。 dev为设备对象实例。

### 函数原型

```lua
function api:del_device(dev)
end
```

## get_device

获取设备对象实例。 此接口对象可以用来读取设备输入项数据，写入设备输出项，发送设备控制项。
当secret值被指定且与设备源应用中设定的secret值一致时，可获取设备的输入项数据写入的权限。
参考: device:share(secret)接口

### 函数原型

```lua
function api:get_device(sn, secret)
end
```

## send_ctrl

发送应用控制指令。 会调用应用设定的handler.on_ctrl。 如需跟踪结果需要设定on_ctrl_result处理函数

### 函数原型

```lua
function api:send_ctrl(app, ctrl, params)
end
```

## cleanup

接口清理接口（sys接口清理时，会自动调用此接口）

### 函数原型

```lua
function api:cleanup()
end
```

## _dump_comm

*内部接口*

### 函数原型

```lua
function api:_dump_comm(sn, dir, ...)
end
```

## _fire_event

*内部接口*

### 函数原型

```lua
api:_fire_event(sn, level, data, timestamp)
```
