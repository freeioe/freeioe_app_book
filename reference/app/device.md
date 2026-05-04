

---

# 设备对象接口

设备对象具备的接口列表

## mod

修改设备描述项。 参考api:add_device

### 函数原型

```lua
function device:mod(inputs, outputs, commands)
end
```

### 参数说明

* inputs
  设备输入项
* outputs
  设备输出项
* commands
  设备指令

## add

在原有设备描述项基础上增加信息。 参考api:add_device

### 函数原型

```lua
function device:add(inputs, outputs, commands)
end
```

## get_input_prop

获取设备输入项的当前值。

### 函数原型

```lua
function device:get_input_prop(input, prop)
end
```

### 示例

```lua
local value, timestamp, quality = dev:get_input_prop('tag1', 'value')
```

### 参数说明

* input
  输入项名称
* prop
  输入项属性名称

### 返回值

返回属性值、时间戳和质量标识，如果未找到则返回nil

## set_input_prop

写入设备输入项属性值。

### 函数原型

```lua
function device:set_input_prop(input, prop, value, timestamp, quality)
end
```

### 参数说明

* input
  输入项名称
* prop
  输入项属性。其中value是用于采集数据值。
* value
  数据
* timestamp
  时间戳。 默认为当前时间
* quality
  质量戳。默认为0

### 示例代码

```lua
dev:set_input_prop("Temperature", "value", 10)
```

### 返回值

成功返回true，失败返回nil和错误消息

## set_input_prop_batch

批量设置设备输入项属性值

> *** API_VER: 4 ***

### 函数原型

```lua
function device:set_input_prop_batch(...)
end
```

### 参数说明

支持两种格式：
1. 表格式：`{{input, prop, value, timestamp, quality}, ...}`
2. 数组格式：`{input, prop, value, timestamp, quality}, ...`

### 返回值

成功返回true，失败返回nil和错误消息

## set_input_prop_emergency

写入设备输入项属性值(紧急数据，需要尽快传递至云端数据)。

*此接口内部会调用set_input_prop接口，保证云端不处理紧急数据的情况下，也会将数据记录到云端。*

### 函数原型

```lua
function device:set_input_prop_emergency(input, prop, value, timestamp, quality)
end
```

### 参数说明

* input
  输入项名称
* prop
  属性名称
* value
  数据值
* timestamp
  时间戳
* quality
  质量戳

## get_output_prop

获取设备输出项当前输出数据

### 函数原型

```lua
function device:get_output_prop(output, prop)
end
```

### 参数说明

* output
  设备输出项名称
* prop
  属性名称（通常为value)

### 返回值

返回属性值和时间戳

## set_output_prop

写入输出项数据

当本函数返回成功后，如需要跟踪执行结果则需要注册on_output_result钩子函数

### 函数原型

```lua
function device:set_output_prop(output, prop, value, timestamp, priv)
end
```

### 参数说明

* output
  设备输出项名称
* prop
  属性名称（通常为value)
* value
  输出的数据值
* timestamp
  输出请求的时间（默认为网关当前时间)
* priv
  输出请求私有数据（用以跟踪执行结果)

### 返回值

成功返回true，失败返回nil和错误消息

## send_command

发送设备控制指令

当本函数返回成功后，如需要跟踪执行结果则需要注册on_command_result钩子函数。 若无需等待结果，则使用nil作为priv的值

### 函数原型

```lua
function device:send_command(command, param, priv)
end
```

### 参数说明

* command
  设备指令名称
* param
  指令参数
* priv
  指令请求私有数据（用以跟踪执行结果)

### 返回值

成功返回true，失败返回nil和错误消息

## sn

获取当前设备的序列号。

### 函数原型

```lua
function device:sn()
end
```

### 返回值

返回设备序列号字符串

## app_name

获取创建此设备的应用实例名称

> *** API_VER: 4 ***

### 函数原型

```lua
function device:app_name()
end
```

### 返回值

返回应用名称

## list_props

获取设备属性，包含inputs, outputs, commands

### 函数原型

```lua
function device:list_props()
end
```

### 返回值

返回包含设备元数据、输入、输出、命令的表

## list_inputs

获取设备所有输入项数据

> *** API_VER: 4 ***

### 函数原型

```lua
function device:list_inputs(data_callback)
```

### 参数说明

* data_callback
  原型为 ```function data_callback(input_name, prop, value, timestamp, quality)```

## data

获取设备所有输入项数据

> *** API_VER: 4 ***

### 函数原型

```lua
function device:data()
end
```

### 返回值

返回以input name为key的table，值包含: value, timestamp, quality属性

## cov

开启设备本地变化发布功能，仅当数据有了变化，才发布给FreeIOE以及其他应用。

> *** API_VER: 4 ***

### 函数原型

```lua
function device:cov(opt)
```

### 参数说明

* opt
  COV选项表或nil禁用COV
  * float_threshold: 浮点数据变化最小值，默认为0.000001
  * ttl: 当数据没有变化时，周期发布的时间间隔。 默认不开启
  * min_ttl_gap: 最小ttl检测周期，默认为10，单位是百分之一秒

## flush_data

强制将当前最新数据遍历并发布到FreeIOE和其他应用

> *** API_VER: 4 ***

### 函数原型

```lua
function device:flush_data()
end
```

## dump_comm

记录设备报文。 参考api:_dump_comm

### 函数原型

```lua
function device:dump_comm(dir, ...)
end
```

### 参数说明

* dir
  方向（send/recv）
* ...
  通信数据

## fire_event

记录设备事件。 参考api:_fire_event

### 函数原型

```lua
function device:fire_event(level, type_, info, data, timestamp)
end
```

### 参数说明

* level
  事件严重级别
* type_
  事件类型字符串
* info
  事件描述
* data
  可选的事件数据表
* timestamp
  可选的事件时间戳

## stat

获取数据统计对象。创建并返回设备统计对象

### 函数原型

```lua
function device:stat(name)
end
```

### 参数说明

* name
  统计名称（例如：packets_in, bytes_out）

### 返回值

返回统计对象

## cleanup

设备清理接口。*此接口为内部接口，无需主动调用*。

### 函数原型

```lua
function device:cleanup()
end
```

## share

设定设备共享密钥，知晓此密钥的其他应用可以获取设备输入项数据的写入权限。

参考：api:get_device(sn, secret)接口

> *** API_VER: 5 ***

### 函数原型

```lua
function device:share(secret)
end
```

### 参数说明

* secret
  密钥字符串，nil表示禁用共享
