
---

# 开发指南


## APP组成

本章介绍如何使用[Lua5.3](http://www.lua.org/manual/5.3/)语言开发FreeIOE应用，在开始阅读本章前，请先熟悉Lua中模块的概念以及如何构建简单的Lua模块。  
 一个经典的APP会有如下的结构：  
 ├── opcua\_client -------- ***App目录***  
 │----├── app.lua --------- ***App入口Lua文件***  
 │----├── conf.lua -------- ***App自定义模块文件***  
 │----└── luaclib --------- ***App自定义的C模块目录***  
 │----------└── opcua.so -- ***App自定义的OpcUA模块（C语言模块）***

APP应用的入口是一个符合FreeIOE框架接口定义的特定Lua模块文件


## 连接设备

### TCP 套接字

FreeIOE框架提供两种TCP Socket连接方式:
1. [SocketChannel](https://github.com/cloudwu/skynet/wiki/socketchannel)
> Skynet 框架提供的TCP Socket通讯框架。有两种工作模式:
> * 同步模式
> * 异步模式(需要协议支持Session)
2. [app.socket](../app/socket.md)
> FreeIOE 封装的简易TCP Socket模式。
3. [Skynet Socket模块](https://github.com/cloudwu/skynet/wiki/socket)

### 连接设备(串口)

FreeIOE 集成了[librs232](http://github.com/srdgame/librs232)模块，支持用户访问串口设备。可选择:
1. SerialChannel
> 同SocketChannel模式的通讯框架
2. [app.serial](../app/serial.md)
> FreeIOE 封装的建议串口模块
3. [rs232](https://github.com/srdgame/librs232/blob/master/bindings/lua/rs232.lua)
> 直接使用librs232模块


### UDP 套接字
TODO:


## 创建设备模型(实例)

在开始真正采集设备数据之前，需要创建设备模型以及设备实例:

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


### 设备描述

设备描述包含:元信息，输入项，输出项，指令集

#### 元信息(meta)

设备元信息包含以下内容:
1. name: 设备描述名称(云平台唯一名称, 如: Symtech S5500)
2. description: 设备描述
3. inst: 设备实例名称(用户可见名称如: 1# S5500电表
4. series: 设备型号（如: v2)


#### 输入项(input)

设备输入项，即从实际传感器设备采集到的数据项，如A相电压、有功总功率等等。

```lua
local inputs = {}
table.insert(inputs, { 
	name='Va',
	desc='A相电压',
})
table.insert(inputs, { 
	name='Mode',
	desc='工作模式',
	vt = 'int', -- 数据类型:整数数值。可以是: float/int/string,缺省为float浮点类型（double)
})
```


#### 输出项(output)

设备输出项，即可以用来调整传感器设备输出的数据项。如：当前输出电压，输出功率等等。

```lua
local outputs = {}
table.insert(outputs, { 
	name='v_output',
	desc='输出电压',
})
```


#### 指令集(command)

设备支持的动作指令。如：开启，停止，暂停等等。
```lua
local commands = {}
table.insert(commands, { 
	name='start',
	desc='启动设备（无需参数)',
})
```


### 设定数据

当通过设备通讯协议收到设备数据项后，设定设备输入项:
```lua
dev:set_input_prop('Va', "value", math.tointeger(v), now, 0)
```


### 响应输出

在api:set_handler的处理对象中增加下面的处理函数。当平台的设备输出请求到达网关后，此处理函数就会被调用。

```lua
--- 处理设备输出项数值变更消息
-- @param app 应用对象实体
-- @param sn 目标设备序列号
-- @param output 输出项名称
-- @param prop 输出项属性(默认为value)
-- @param value 输出数值
on_output = function(app, sn, output, prop, value)
	-- You code here
end,
```


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


### 紧急数据

紧急数据是一种需要FreeIOE尽快将之传输至平台的数据，设定方式为:
```lua
dev:set_input_prop_emergency('Va', "value", math.tointeger(v), now, 0)
```
> 紧急数据将绕过一系列的上传检测/打包等操作，请谨慎使用。


### 设备事件

当设备/应用出现异常，或者任何其他需要通知用户的可读信息，通过设备的事件接口将信息上传至平台。

```lua
	local event = require 'app.event'
	local info = "/tmp disk is nearly full!!!"
	dev:fire_event(event.LEVEL_ERROR, event.EVENT_SYS, info, {used=10000000, free=0})
```


### 通讯报文

当从链路上接受到数据时，可以通过通讯报文接口进行输出。

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


### 通讯统计

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


### 应用日志

参考[日志接口](../app/logger.md)

```lua
	self._log:trace("read input registers done!")
	self._log:trace("Got err:", err, "more", "log content", here)
```


## 其他


### 设备序列号

如设备协议中并未提供设备的序列号读取方式、或者不想采用设备本身的序列号，则可以使用FreeIOE提供的序列号生成功能，生成序列号。参考：

```lua
local dev_sn = self._sys:gen_sn('S5500_#1')
```

如需更为简单的序列号方式，则可以使用网关序列号 + 应用示例名 + 设备序号:

```lua
local dev_sn = self._sys:id()..self._name..'#1'
```


### 云配置

云平台提供应用的云配置服务，可以存储应用配置、设备模板等文本信息。在应用中可以使用sys:conf_api来获取接口，从而从平台下载配置的内容。

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


## 应用参考

FreeIOE 在Github上提供一些示例应用： [代码库](https://github.com/freeioe/freeioe_example_apps)



