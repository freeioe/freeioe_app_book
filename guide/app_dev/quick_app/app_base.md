
---

# 采集应用

此章我们将简单介绍如何开发一个设备数据采集的 FreeIOE 应用。

> 数据采集应用通常用来通过某种通讯协议获取网关连接的设备中的传感器数据，并将数据发布到 FreeIOE 网关内，从而数据上云应用以及边缘计算类应用可以使用这些数据。

此应用示例具备了：

1. 创建设备模型
2. 发布设备模型数据
3. 输出应用日志
4. 输出设备通讯报文
5. 输出设备统计数据

[示例代码](#示例代码)

## 创建设备模型

数据采集类应用会向 FreeIOE 发布设备数据，因此需要在应用启动时，创建设备模型以及实例。

### 设备序列号

每一个具体的设备实例在平台内，都需要一个唯一的设备序列号，可以使用基础模块的gen_sn函数来生成。

### 设备描述

设备模型的描述包含

* 元信息
* 输入项
* 输出项
* 指令集

#### 元信息(meta)

设备元信息包含以下内容:

1. name
   设备名称(云平台唯一名称, 如，某某锅炉，某某电表
2. description
   为设备产品的具体描述信息，如某某品牌三相交流智能电表
3. inst
   为选填属性，如 1#锅炉，3#电表， 办公室空调一号等。 是标记设备的友好名称
4. series
   为设备产品的系列号，如 S5500, S7-300
5. manufacturer
   设备厂家信息（如：冬笋科技（北京）有限公司)
6. link
   选填，设备详细信息的链接地址, 默认会是 ```http://device.freeioe.org/devices?name=<meta.name>```

您可在元信息中附加更多的静态信息。请勿使用meta来标记一些动态信息，FreeIOE 在设备实例创建、修改、删除时，会将网关内所有设备信息全量同步到平台。滥用元信息会导致流量浪费和平台的负载增加。

类似元信息，输入项、输出项以及指令集都可以自定义静态属性。

#### 输入项(inputs)

设备输入项是从实际传感器设备采集到的数据项，如A相电压、有功总功率等等。 也可以是计算出来的数据项，如当前4G数据发送总量、当日产量等等。只有输入项的数据、才可以进行数据发布。

```lua
local inputs = {}
table.insert(inputs, {
	name='Va',
	unit='V',
	desc='输出电压',
	vt='float',
})
```

##### name

输入项的名称，需要设备内唯一。

##### desc

输入项的描述信息。

#### vt

输入项发布数据的数值类型，可以是：

* float
  双精度浮点类型
* int
  具体限制参考 FreeIOE 编译时定义的整数类型，通常为64位整数
* string
  字符串类型

> 数据发布时，FreeIOE 会自动进行数据的验证以及转换，请务必设定为正确的数据类型，确保数据的一致性和严禁性。

#### 输出项(outputs)

设备输出项是指设备具备可以对外输出的能力，模拟量输出、开关量输出、可调节输出电压等等。备输出项的设备，用户就可从而平台可以进行设备数据下置。

> 其中下置的参数包含要输出的数据, 通常为字符串，应用需要自己做一下数据转换

```lua
local outputs = {}
table.insert(outputs, {
	name='Voutput1',
	unit='V',
	desc='输出电压',
})
```

#### 指令集(commands)

当设备支持的执行一些动作指令，如开启，停止，暂停，可以在设备模型中声明这些动作指令。

> 平台在下发指令时附带一个param的数据结构，在lua里面是一个table的对象。 不过FreeIOE并不会进行参数转换以及验证的工作，应用需要正确处理传入的参数的严禁性以及合法性验证。

```lua
local commands = {}
table.insert(commands, {
	name='start',
	desc='启动设备（无需参数)',
})
```

## 发布设备数据

当通过设备通讯协议收到设备数据项的新的数值后，请使用下面接口进行设备数据发布:

```lua
dev:set_input_prop('Va', "value", val, now, 0)
```

其中：
val 为协议中解析出来的具体数据，类型请保持和模型声明中的vt一致
now 是数据的时间戳，如协议中没有时间戳（如modbus），请使用 FreeIOE 提供的系统时间

## 发布紧急数据

设备紧急数据是一种需要 FreeIOE 尽快将之传输至平台的设备数据。

发布紧急数据的调用:

```lua
dev:set_input_prop_emergency('Va', "value", val, now, 0)
```

> 紧急数据将绕过一系列的上传检测/打包等操作，请谨慎使用。在冬笋平台下，此种数据会优先/即时传输至平台，保证数据的时效性。
> 请勿滥用此紧急数据接口，因为此接口会同步发布正常的数据，即一次数据会被传输两次（不同通道），滥用此接口会导致平台压力增加和流量使用的增加。

## 发布通讯报文

当应用支持平台查看通讯报文时，需要使用此通讯报文发布接口。

```lua
--- 设定通讯口数据回调
client:set_io_cb(function(io, msg)
	--- 输出通讯报文
	dev:dump_comm(io, msg)
	--- 计算统计信息
	if io == 'IN' then
		stat:inc('bytes_in', string.len(msg))
	else
		stat:inc('bytes_out', string.len(msg))
	end
end)
```

详细使用可以参考应用示例库中的/modbus/master应用。

## 发布通讯统计

通讯统计信息，如接收包数，发送包数，接收字节数，发送字节数等等。

统计类型：

* status
* success_ratio
* error_ratio
* packets_in
* packets_out
* packets_error
* bytes_in
* bytes_out
* bytes_error


```lua
stat:inc('packets_in', 1)
```

## 输出应用日志

参考[日志接口](../../../reference/app/logger.md)

```lua
self._log:trace("read input registers done!")
self._log:trace("Got err:", err, "more", "log content", here)
```

## 其他

### 云配置获取

云平台提供应用的云配置服务，可以存储应用配置、设备模板等文本信息。从而在应用中可以使用sys:conf_api从平台获取这些数据。

```lua
local api = self._sys:conf_api('TPL000000001')
local ver = api:version()

local str = api:data(ver)

-- If template is cjson format
local conf = cjson.decode(str)

-- If template is csv
local conf = ftcsv.parse(str)
```

### 响应输出(数据下置)

在api:set_handler的处理对象中增加下面的处理函数。当平台的设备输出请求到达网关后，此处理函数就会被调用。

```lua
--- 处理设备输出项数值变更消息
-- @param app 应用对象实体
-- @param sn 目标设备序列号
-- @param output 输出项名称
-- @param prop 输出项属性(默认为value)
-- @param value 输出数值
-- @param timestamp 数据源的时间戳，选项数据，可用于校验下置数据是否超过了时效。
on_output = function(app, sn, output, prop, value, timestamp)
	-- You code here
end,
```

本函数需要正确反馈执行结果，执行成功返回true, 失败返回false, \<error\>。FreeIOE会自动汇报执行结果到云平台。 此函数可以进行sleep, yield操作。

### 响应指令

在api:set_handler的处理对象中增加下面的处理函数。当平台的设备指令请求到达网关后，此处理函数就会被调用。

```lua
--- 处理设备指令消息
-- @param app 应用对象实体
-- @param sn 目标设备序列号
-- @param command 指令名称
-- @param param 参数(可以是数值，字符串，table)
on_command = function(app, sn, command, param)
	-- You code here
end,
```

本函数如同on_output一样，需要正确返回执行结果。 也可以进行sleep, yield等操作。


### 发布设备事件

当设备/应用出现异常，或者任何其他需要通知用户的事件信息，可以通过设备的事件接口发布。

```lua
	local event = require 'app.event'
	local info = "/tmp disk is nearly full!!!"
	dev:fire_event(event.LEVEL_ERROR, event.EVENT_SYS, info, {used=10000000, free=0})
```

注意请勿频繁调用此接口，建议记录事件发布的时间，确保应用不会无限制的循环发布事件。虽然FreeIOE内置了事件发布的次数限制（20条/分钟)，但是强烈建议应用设定事件发布限制。


## 设备通讯

### TCP 套接字

FreeIOE框架提供TCP Socket连接有以下几种方式:

1. [SocketChannel](https://github.com/cloudwu/skynet/wiki/socketchannel) <br>
Skynet 框架提供的TCP Socket通讯框架。有两种工作模式:<br>
    * 同步模式
    * 异步模式(需要协议支持Session)
2. [app.socket](../app/socket.md)<br>
FreeIOE 封装的简易TCP Socket模式。
3. [Skynet Socket模块](https://github.com/cloudwu/skynet/wiki/socket)

参考示例应用库中的/modbus/master /modbus/slave /modbus/gateway 以及 /other/dtu 和 /example/serial_socket应用


### 串口通讯

FreeIOE 集成了 [librs232](http://github.com/srdgame/librs232) 模块，支持用户访问串口设备。

1. SerialChannel<br>
同SocketChannel模式的通讯框架
2. [app.serial](../app/serial.md)<br>
FreeIOE 封装的建议串口模块
3. [rs232](https://github.com/srdgame/librs232/blob/master/bindings/lua/rs232.lua)<br>
直接使用librs232模块

参考示例应用库中的modbus应用，以及 /other/dtu 和 /other/oliver_355_monitor、example/serial 等应用。


### UDP 套接字

详见 Skynet Socket模块的说明。 并且FreeIOE扩展了Socket中的sendto函数，除了原本的sendto(id, from, data)之外，支持sendto(id, ip, port, data)方式直接指定发送目标的IP和端口信息。

## 示例代码

```lua
local app_base = require 'app.base'

local app = app_base:subclass('THIS_IS_AN_SAMPLE_SUB_APP')
app.static.API_VER = 6 -- 可不设置

---
-- 应用对象初始化回调函数
-- @param name: 应用本地安装名称。 如modbus_com_1
-- @param sys: 系统sys接口对象。参考API文档中的sys接口说明
-- @param conf: 应用配置参数。由安装配置中的json数据转换出来的数据对象
function app:on_init()
	--- 设备实例
	self._devs = {}
	self._log:debug("XXXX Application initialized")
end

--- 应用启动函数
function app:on_start()
	--- 生成设备模型唯一序列号
	local sys_id = self._sys:id()
	local sn = self:gen_sn('example_device_serial_number')

	--- 创建设备模型
	local inputs = {
		{name="tag1", desc="tag1 desc", unit="KV", vt="float"}
	}
	local meta = self._api:default_meta()
	meta.name = "Example Device"
	meta.description = "Example Device Meta"
	--- 生成设备模型对象
	local dev = self._api:add_device(sn, meta, inputs)
	self._dev = dev
	--- 生成设备通讯口统计对象
	local stat = dev:stat('port')
	self._stat = stat

	return true
end

--- 应用退出函数
function app:on_close(reason)
    -- 处理通讯链路关闭等
end

--- 应用运行入口
function app:on_run(tms)
	local dev = self._dev
	local stat = self._stat
	dev:dump_comm("IN", "XXXXXXXXXXXX")
	dev:set_input_prop('tag1', "value", math.random())
	stat:inc('packets_in', 1)

	return 10000 --单位是ms, 10000代表下一采集间隔为10秒。 等同于sleep(10000)
end

--- 返回应用对象(标准模块做法)
return app
```
