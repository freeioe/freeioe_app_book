
---

# 简易应用封装模块


本模块封装了应用的基础逻辑。帮助用户快速开发FreeIOE应用。

示例: [示例应用](https://github.com/freeioe/freeioe_example_apps/blob/master/sample/app.lua)

> *** API_VER: 5 ***

## 模块实现的函数

### initialize
> function app:initialize(name, sys, conf)
>

提供了默认的应用构造函数， 并在此函数最后调用on_init函数

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

### create_calc
> function app:create_calc()
> 

生成并返回一个app.utils.calc模块对象，并赋值到self._calc。 封装模块会正确处理这个对象的初始化以及销毁工作。注意本函数只能在on_init回调里面使用，才能保证正常工作

### get_calc
> function app:get_calc()
>

获取已经生成的calc模块对象


## 模块数据对象成员列表

* \_sys -- FreeIOE系统接口
* \_name -- 应用实例名称
* \_conf -- 应用配置数据对象
* \_api -- FreeIOE系统数据接口
* \_log -- FreeIOE系统日志接口
* \_calc -- FreeIOE系统提供的计算帮助对象,需要在on_init中使用create_calc初始化


## 如何使用此封装模块

使用此封装模块时，只需要按需在应用对象里面实现自己需要的回调函数即可。模块会正确识别并向FreeIOE系统注册钩子函数。 

### 应用运行相关函数

* on_init
* on_start
* on_close
* on_run (optional)


### 监听设备数据相关函数

如需处理设备数据(其他应用获取的设备数据），如实现数据上送、数据计算等，则需在基于本模块派生的应用模块里面实现以下函数

* on_add_device -- 设备新增
	> function app:on_add_device(src_app, sn, props)
* on_mod_device -- 设备修改
	> function app:on_mod_device(src_app, sn, props)
* on_del_device -- 设备移除
	> function app:on_del_device(src_app, sn)
* on_input -- 设备输入项数据
	> function app:on_input(src_app, sn, input, prop, value, timestamp, quality)
* on_input_em -- 设备输入项紧急数据（如无需特别处理，可不实现此函数，此紧急数据还会继续调用on_input)
	> function app:on_input_em(src_app, sn, input, prop, value, timestamp, quality)
* on_comm -- 设备报文
	> function app:on_comm(src_app, device_sn, dir, timestmap, ...)
* on_stat -- 设备统计数据
	> function app:on_stat(src_app, device_sn, stat, prop, value timestmap)
* on_event -- 设备事件信息
	> function app:on_event(src_app, device_sn, event_level, event_data, event_timestamp)


### 设备输出、指令相关函数

* on_output -- 向本应用创建的设备请求设备数据输出
	> function app:on_output(src_app, sn, output, prop, value, timestamp)
* on_command -- 向本应用创建的设备请求执行设备指令
	> function app:on_command(src_app, sn, command, params)

### 应用控制交互函数

* on_ctrl -- 应用间进行控制交互，参考:app.api.send_ctrl
	> function app:on_ctrl(src_app, command, param)

### 设备输出、指令以及应用控制交互结果函数

* on_output_result -- 处理本应用发起的设备输出请求的执行结果
	> function app:on_output_result(src_app, priv, result, err)
* on_command_result -- 处理本应用发起的设备指令请求的执行结果
	> function app:on_command_result(src_app, priv, result, err)
* on_ctrl_result -- 处理本应用发起的应用控制交互的执行结果
	> function app:on_ctrl_result(src_app, priv, result, err)


