
---

# 使用基础类构建应用

前面我们介绍一个从零开始的应用结构和代码，当你去构建第二应用时，会发现这些代码复制现象太严重。而 FreeIOE 也提供了应用的基础类。

应用基础类提供了什么：

* 使用 middleclass 构造了应用类生命
* 实现了 FreeIOE 要求的接口函数
* 在实现了的接口函数的回调，例如 initialize 函数会尝试回调 on_init函数
* 封装了应用配置可视化 JSON 文件的解读，并将其实中的缺省值作为默认值传递给应用
* 注册了系统的回调函数，并映射至应用自身的成员函数上
* 提供数据计算的入口

使用基础类后的示例应用代码如下：


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
	self._log:debug("XXXX Application initlized")
end

--- 应用启动函数
function app:on_start()
	--- 生成设备唯一序列号
	local sys_id = self._sys:id()
	local sn = self:gen_sn('example_device_serial_number')

	--- 增加设备实例
	local inputs = {
		{name="tag1", desc="tag1 desc", unit="KV"}
	}
	local meta = self._api:default_meta()
	meta.name = "Example Device"
	meta.description = "Example Device Meta"
	local dev = self._api:add_device(sn, meta, inputs)
	self._devs[#self._devs + 1] = dev

	return true
end

--- 应用退出函数
function app:on_close(reason)
    -- 处理通讯链路关闭等
end

--- 应用运行入口
function app:on_run(tms)
	for _, dev in ipairs(self._devs) do
		dev:dump_comm("IN", "XXXXXXXXXXXX")
		dev:set_input_prop('tag1', "value", math.random())
	end

	return 10000 --单位是ms, 10000代表下一采集间隔为10秒。 等同于sleep(10000)
end

--- 返回应用对象(标准模块做法)
return app
```

## 回调函数列表

### on_init

应用构造回调函数，当你需要在应用构造做一些处理，则实现这个函数。

### on_start

应用启动回调，实现您应用自身逻辑的函数

### on_close

应用停止/推出函数回调

### on_run

周期运行回调函数

### on_add_device(src_app, sn, props)

如果需要关注其他应用注册的设备模型，那么可以实现次函数。基础类在接受模型注册消息时会调用此函数

### on_mod_device(src_app, sn, props)

如果需要关注其他应用注册的设备模型，那么可以实现次函数。基础类在接受模型注册消息时会调用此函数

### on_del_device(src_app, sn)

如果需要关注其他应用注册的设备模型，那么可以实现次函数。基础类在接受模型注册消息时会调用此函数

### on_input(src_app, sn, input, prop, value, timestamp, quality)

接收设备模型的数据发布

### on_input_em(src_app, sn, input, prop, value, timestamp, quality)

接收设备模型发布的紧急数据

### on_output(src_app, sn, output, prop, value, timestamp, priv)

设备数据下置请求函数，如支持下置数据到设备。注册设备模型包含有下置数据的信息，然后在此函数实现设备数据下置。

### on_output_result(src_app, priv, result, err)

如果应用请求了其他应用创建的设备模型中的数据下置，该数据下置被其他应用执行后，FreeIOE 会将结果传递给此函数

### on_command(src_app, sn, command, param, priv)

如果应用创建的设备模型具备指令集，需要实现此函数来完成设备指令动作

### on_command_result(src_app, priv, result, info)

如果应用请求了其他应用创建的设备模型中的指令，该指令被其他应用执行后，FreeIOE 会将结果传递给此函数

### on_ctrl(src_app, command, param, priv)

应用之间可以进行一些非设备模型依赖的数据交换，行为控制。实现此函数可接收来自其他应用的请求

### on_ctrl_result(src_app, priv, result, info)

如果对其他应用发出了数据交换、行为控制，在对方执行完成后，FreeIOE 会回调此函数通知执行结果

### on_comm

如果应用关注其他应用产生的设备交互报文数据，那么请实现此函数

### on_stat

如果应用关注其他应用产生的设备统计数据，那么请实现此函数

### on_event

如果应用关注其他应用或系统产生的事件，那么请实现此函数

## 计算接口

计算接口、只是由 FreeIOE 提供的帮助应用去监听设备模型中的数据发布，并在合适的时机来调用执行您注册的执行代码

### create_calc

创建计算接口。此接口只能在on_init里面进行调用，没有返回值

### get_calc

获取计算接口实例对象

## 其它

### gen_sn(key)

使用唯一的key（网关内唯一），叠加网关序列号来生成全局（全平台）唯一设备序列号