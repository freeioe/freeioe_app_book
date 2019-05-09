
---

# 简易应用封装模块


本模块封装了应用的基础逻辑。帮助用户快速开发FreeIOE应用。


## 封装的函数

### initialize
> function app:initialize(name, sys, conf)
>

提供了默认的应用构造函数

### start

提供默认的应用启动函数，并在此函数最后调用on_start函数

### close

提供默认的应用停止函数，并在此函数最后调用on_close函数

### run

提供默认的应用运行函数，如应用提供了on_run函数，则会使用on_run的函数逻辑


### gen_sn
> function app:gen_sn(key)
> 

生成基于FreeIOE网关序列号的设备序列号，格式: <gateway_sn>.<hashed_key_string>


## 如何使用此封装模块

### 提供应用运行的回调函数

* on_start
* on_close
* on_run (optional)


### 处理设备数据接口回调

如需处理设备数据(其他应用获取的设备数据），如实现数据上送、数据计算等，则需在基于本模块派生的应用模块里面实现以下函数

* on_add_device
	> 如需处理设备信息数据，则实现此接口: function app:on_add_device(src_app, sn, props)
* on_mod_device
	> 如需处理设备信息数据，则实现此接口: function app:on_mod_device(src_app, sn, props)
* on_del_device
	> 如需处理设备信息数据，则实现此接口: function app:on_del_device(src_app, sn)
* on_input
	> 如需处理设备数据，则实现此接口: function app:on_input(src_app, sn, input, prop, value, timestamp, quality)
* on_input_em
	> 如需处理设备紧急数据，则实现此接口: function app:on_input_em(src_app, sn, input, prop, value, timestamp, quality)
* on_comm
	> 如需处理设备通讯报文，则实现此接口: function app:on_comm(app_src, device_sn, dir, timestmap, ...)
* on_stat
	> 如需处理通讯统计数据，则实现此接口: function app:on_stat(app_src, device_sn, stat, prop, value timestmap)
* on_event
	> 如需处理设备事件信息，则实现此接口: function app:on_event(app_src, device_sn, event_level, event_data, event_timestamp)


### 提供设备输出、指令

* on_output
	> function app:on_output(app_src, sn, output, prop, value, timestamp)
* on_command
	> function app:on_command(app_src, sn, command, params)

### 应用控制交互

* on_ctrl
	> function app:on_ctrl(app_src, command, param)

### 设备输出、指令以及应用控制交互结果

* on_output_result
	> function app:on_output_result(app_src, priv, result, err)
* on_command_result
	> function app:on_command_result(app_src, priv, result, err)
* on_ctrl_result
	> function app:on_ctrl_result(app_src, priv, result, err)


