
----

# 基础接口

FreeIOE框架为每个应用创建的服务接口，用以帮助应用快速构建设备模型等操作


### set_handler
> function api:set_handler(handler, watch_data)

设定处理函数。

* handler：接口对象
* watch_data: 是否关注其他应用创建的设备数据消息


> 示例:
```
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
```

备注:
* src_app -- 消息源应用的实例名(string)
* dev_sn -- 设备序列号(string)


### list_devices
> function api:list_devices()

枚举系统中所有设备对象的描述信息 (meta, inputs, outputs, commands等等)


### add_device
> function api:add_device(sn, meta, inputs, outputs, commands)

创建新的采集设备对象。返回设备对象实例（参考[设备API](device.md))。

* sn：设备序列号
* meta: 设备Meta信息
* inputs：设备输入项列表
* outputs：设备输出项列表
* commands：设备控制项列表


### del_device
> function api:del_device(dev)

删除设备。 dev为设备对象实例。


### get_device
> function api:get_device(sn, secret)

获取设备对象实例。 此接口对象可以用来读取设备输入项数据，写入设备输出项，发送设备控制项。
当secret值被指定且与设备源应用中设定的secret值一致时，可获取设备的输入项数据写入的权限。
参考: device:share(secret)接口


### send_ctrl
> function api:send_ctrl(app, ctrl, params)

发送应用控制指令。 会调用应用设定的handler.on_ctrl。 如需跟踪结果需要设定on_ctrl_result处理函数


### cleanup
> function api:cleanup()

接口清理接口（sys接口清理时，会自动调用此接口）

### api:\_dump_comm(sn, dir, ...)

*内部接口*


### api:\_fire_event(sn, level, data, timestamp)

*内部接口*
