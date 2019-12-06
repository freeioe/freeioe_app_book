
---

# 设备对象接口

设备对象具备的接口列表

## mod

修改设备描述项。 参考api:create_device

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

在原有设备描述项基础上增加信息。 参考api:create_device

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

### 参数说明

* input
  输入项名称
* prop
  输入项属性名称

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

## sn

获取当前设备的序列号。

> *** API_VER: 5 ***

### 函数原型

```lua
function device:sn()
end
```

## list_props

获取设备属性，包含inputs, outputs, commands

### 函数原型

```lua
function device:list_props()
end
```

## list_inputs

获取设备所有输入项数据

> *** API_VER: 4 ***

### 函数原型

```lua
> function device:list_inputs(data_callback)
```

### 参数说明

* data_callback
  原型为 ```function data_callback(input_name, prop, value, timestamp, quality)```

## data

获取设备所有输入项数据，

> *** API_VER: 4 ***

```lua
function device:data(opt)
end
```

返回结果是以input name为key的table，值包含: value, timestamp, quality属性

## cov

开启设备本地变化发布功能，仅当数据有了变化，才发布给FreeIOE以及其他应用。

> *** API_VER: 4 ***

### 函数原型

```lua
> function device:cov(opt)
```

### 参数说明

* opt
  * float_threshold: 浮点数据变化最小值，默认为0.000001
  * ttl: 当数据没有变化时，周期发布的时间间隔。 默认不开启
  * min_ttl_gap: 最小ttl检测周期，默认为10，单位是百分之一秒

## flush_data

强制将当前最新数据遍历并发布到FreeIOE和其他应用

> *** API_VER: 4 ***

### 函数原型

```lua
function device:flush_data(opt)
end
```

## dump_comm

记录设备报文。 参考sys:dump_comm

### 函数原型

```lua
function device:dump_comm(dir, ...)
end
```

## fire_event

记录设备事件。 参考sys:fire_event

### 函数原型

```lua
> function device:fire_event(level, type, info, data, timestamp)
```

## stat

获取数据统计对象。参考app:stat

### 函数原型

```lua
function device:stat(name)
end
```

## cleanup

设备清理接口。*此接口为内部接口，无需主动调用*。

### 函数原型

```lua
function device:cleanup()
end
```

## share

### 函数原型

设定设备共享密钥，知晓此密钥的其他应用可以获取设备输入项数据的写入权限。

参考：api:get_device(sn, secret)接口

> *** API_VER: 5 ***

```lua
function device:share(secret)
end
```