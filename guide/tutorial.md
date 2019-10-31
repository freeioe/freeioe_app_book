
---

# 应用开发指南


## 创建设备模型(实例)

在开始进行设备通讯处理设备数据之前，需要创建设备模型以及设备实例。创建设备实例之后才能进行设备数据更新(发布)

```lua
	--- 生成设备的序列号 
	local dev_sn = "this_is_device_serial_number"
	local meta = self._api:default_meta()
	meta.name = "Modbus"
	meta.description = "Modbus Device"
	meta.series = "S5500"
	--- 生成设备模型对象
	local dev = self._api:add_device(dev_sn, meta, inputs, outputs, commands)
	--- 生成设备通讯口统计对象
	local stat = dev:stat('port')
```

注意：

* 设备序列号需要保证数据平台内部唯一，建议使用网关序列号作为前缀，从而避免设备序列号重复导致的数据错乱的问题。 请参考示例应用库中的处理方式
* 设备元信息(meta)中
    * name 为设备名称，如，某某锅炉，某某电表 为产品名称)
    * inst 为选填属性，如 1#锅炉，3#电表， 办公室空调一号等。 是标记设备的友好名称
    * description 为设备产品的具体描述信息，如某某品牌三相交流智能电表
    * series 为设备产品的系列号，如 S5500, S7-300
    * manufacture


### 设备描述

设备描述包含:元信息，输入项，输出项，指令集

#### 元信息(meta)

设备元信息包含以下内容:
1. name: 设备描述名称(云平台唯一名称, 如: Symtech S5500)
2. description: 设备描述
3. inst: 设备实例名称(用户可见名称如: 1# S5500电表
4. series: 设备型号（如: v2)
5. manufacturer: 设备厂家信息（如：冬笋科技（北京）有限公司)
6. link: 设备详细信息的链接地址 （如： http://device.freeioe.org/devices?name=T1-300)

用户可在meta信息中附加自己需要的静态信息。注意FreeIOE只有在设备实例创建、修改、删除时才会将网关内所有设备信息全量同步到平台。请勿滥用元信息，导致无畏的流量占用。

类似metai信息，input/output/command除了本文档定义的属性外，也是可以自定义属性，注意事项如同meta信息。


#### 输入项(input)

设备输入项，可以是从实际传感器设备采集到的数据项，如A相电压、有功总功率等等。 也可以是计算出来的数据项，如当前4G数据发送总量、当日产量等等。

```lua
local inputs = {}
table.insert(inputs, {
	name='Va',
	unit='V',
	desc='A相电压',
})
table.insert(inputs, {
	name='Mode',
	desc='工作模式',
	vt = 'int', -- 数据类型:整数数值。
})
```

数据类型有：

* float
* int
* string

缺省为float浮点类型（double)，其中int为lua的整数类型，具体限制参考FreeIOE编译时定义的整数类型，通常为64位整数。 FreeIOE会自动进行数据的验证以及转换，请务必设定为正确的数据类型，确保数据的一致性和严禁性。


#### 输出项(output)

设备输出项，即可以用来调整传感器设备输出的数据项。如：当前输出电压，输出功率等等。平台在有output的设备上可以进行数据下置，其中下置的参数包含要输出的数据(通常为字符串，应用需要自己做一下数据转换)

```lua
local outputs = {}
table.insert(outputs, {
	name='v_output',
	unit='V',
	desc='输出电压',
})
```


#### 指令集(command)

设备支持的动作指令。如：开启，停止，暂停等等。 平台在下发指令时附带一个param的数据结构，在lua里面是一个table的对象。 不过FreeIOE并不会进行参数转换以及验证的工作，应用需要正确处理传入的参数的严禁性以及合法性验证。
```lua
local commands = {}
table.insert(commands, {
	name='start',
	desc='启动设备（无需参数)',
})
```


### 发布设备数据

当通过设备通讯协议收到设备数据项后，使用下面接口进行设备数据发布:
```lua
dev:set_input_prop('Va', "value", val, now, 0)
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


### 发布紧急数据

设备紧急数据是一种需要FreeIOE尽快将之传输至平台的数据，发布方式为:
```lua
dev:set_input_prop_emergency('Va', "value", val, now, 0)
```
> 紧急数据将绕过一系列的上传检测/打包等操作，请谨慎使用。在冬笋平台下，此种数据会优先/即时传输至平台，保证数据的时效性。
> 请勿滥用此紧急数据接口，因为此接口会同步发布正常的数据，即一次数据会被传输两次（不同通道），滥用此接口会导致平台压力增加和流量使用的增加。


### 发布设备事件

当设备/应用出现异常，或者任何其他需要通知用户的事件信息，可以通过设备的事件接口发布。

```lua
	local event = require 'app.event'
	local info = "/tmp disk is nearly full!!!"
	dev:fire_event(event.LEVEL_ERROR, event.EVENT_SYS, info, {used=10000000, free=0})
```

注意请勿频繁调用此接口，建议记录事件发布的时间，确保应用不会无限制的循环发布事件。虽然FreeIOE内置了事件发布的次数限制（20条/分钟)，但是强烈建议应用设定事件发布限制。


### 发布通讯报文

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


### 发布通讯统计

通讯统计信息，如接收包数，发送包数，接收字节数，发送字节数等等。

* 统计类型
	* status
	* success_ratio
	* error_ratio
	* packets_in
	* packets_out
	* packets_error
	* bytes_in
	* bytes_out
	* bytes_error


代码示例，见通讯报文代码示例


### 输出应用日志

参考[日志接口](../app/logger.md)

```lua
	self._log:trace("read input registers done!")
	self._log:trace("Got err:", err, "more", "log content", here)
```


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

FreeIOE 集成了[librs232](http://github.com/srdgame/librs232)模块，支持用户访问串口设备。

1. SerialChannel<br>
同SocketChannel模式的通讯框架
2. [app.serial](../app/serial.md)<br>
FreeIOE 封装的建议串口模块
3. [rs232](https://github.com/srdgame/librs232/blob/master/bindings/lua/rs232.lua)<br>
直接使用librs232模块

参考示例应用库中的modbus应用，以及 /other/dtu 和 /other/oliver_355_monitor、example/serial 等应用。


### UDP 套接字

详见 Skynet Socket模块的说明。 并且FreeIOE扩展了Socket中的sendto函数，除了原本的sendto(id, from, data)之外，支持sendto(id, ip, port, data)方式直接指定发送目标的IP和端口信息。


## 其他


### 设备序列号生成

FreeIOE要求有所设备对象必须有自己的唯一的序列号，此唯一不但是指网关设备内唯一，而且要求保证在平台内唯一。

如设备协议中并未提供设备的序列号读取方式、或者不想采用设备本身的序列号，则可以使用FreeIOE提供的序列号生成功能，生成序列号。参考：

```lua
local dev_sn = self._sys:gen_sn('S5500_#1')
```

如需更为简单的序列号方式，则可以使用网关序列号 + 应用示例名 + 设备序号:

```lua
local dev_sn = self._sys:id()..self._name..'#1'
```


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


## 社区

访问FreeIOE [应用开发](http://app.freeioe.org)


## 示例应用库

FreeIOE 在Github上提供一些示例应用： [代码库](https://github.com/freeioe/freeioe_example_apps)



