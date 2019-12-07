
---

# 应用基础类模块

> ***API_VER: 5***

本模块封装了应用的基础逻辑。帮助用户快速开发FreeIOE应用。

应用基础类提供了：

* 使用 middleclass 构造了面向对象基础类
* 实现了 FreeIOE 应用必备的接口函数
* 在实现了的接口函数的回调，例如 initialize 函数会尝试回调 on_init函数
* 封装了应用配置可视化 JSON 文件的解读，并将其实中的缺省值作为默认值传递给应用
* 注册了系统的回调函数，并映射至应用自身的成员函数上
* 提供数据计算的入口

# 成员函数

## gen_sn

```lua
function app:gen_sn(key)
end
```

使用唯一的key（网关内唯一），叠加网关序列号来生成全局（全平台）唯一设备序列号 格式: <gateway_sn>.<hashed_key_string>

## create_calc

生成并返回一个```app.utils.calc``` 模块对象，并赋值到 ```self._calc```。 封装模块会正确处理这个对象的初始化以及销毁工作。注意本函数只能在on_init回调里面使用，才能保证正常工作。

```lua
function app:create_calc()
end
```

## get_calc

获取已经生成的calc模块对象

```lua
function app:get_calc()
```

# 成员对象

* \_sys
  FreeIOE 系统接口
* \_name
  应用实例名称
* \_conf
  应用配置数据对象
* \_api
  FreeIOE 系统数据接口
* \_log
  FreeIOE 系统日志接口
* \_calc
  FreeIOE系统提供的计算帮助对象,需要在on_init中使用create_calc初始化

# 如何使用

使用此封装模块时，只需要按需在应用对象里面实现自己需要的回调函数即可。模块会正确识别并向FreeIOE系统注册钩子函数。

## 应用运行相关函数

* on_init
  应用构造回调函数，当你需要在应用构造做一些处理，则实现这个函数。
* on_start
  应用启动回调，实现您应用自身逻辑的函数
* on_close
  应用停止/推出函数回调
* on_run (optional)
  周期运行回调函数

## 监听设备数据相关函数

如需处理设备数据(其他应用获取的设备数据），如实现数据上送、数据计算等，则需在基于本模块派生的应用模块里面实现以下函数

* on_add_device
  如果需要关注其他应用注册的设备模型，那么可以实现次函数。基础类在接受模型注册消息时会调用此函数。
  ```lua
  function app:on_add_device(src_app, sn, props)
  ```
* on_mod_device
  如果需要关注其他应用注册的设备模型，那么可以实现次函数。基础类在接受模型注册消息时会调用此函数
  ```lua
  function app:on_mod_device(src_app, sn, props)
  ```
* on_del_device
  如果需要关注其他应用注册的设备模型，那么可以实现次函数。基础类在接受模型注册消息时会调用此函数
  ```lua
  function app:on_del_device(src_app, sn)
  ```
* on_input
  接收设备模型的数据发布
  ```lua
  function app:on_input(src_app, sn, input, prop, value, timestamp, quality)
  ```
* on_input_em
  接收设备模型发布的紧急数据（如无需特别处理，可不实现此函数，此紧急数据还会继续调用on_input)
  ```lua
  function app:on_input_em(src_app, sn, input, prop, value, timestamp, quality)
  ```
* on_comm
  如果应用关注其他应用产生的设备交互报文数据，那么请实现此函数
  ```lua
  function app:on_comm(src_app, device_sn, dir, timestamp, ...)
  ```
* on_stat -- 设备统计数据
  如果应用关注其他应用产生的设备统计数据，那么请实现此函数
  ```lua
  function app:on_stat(src_app, device_sn, stat, prop, value timestamp)
  ```
* on_event
  如果应用关注其他应用或系统产生的事件，那么请实现此函数
  ```lua
  function app:on_event(src_app, device_sn, event_level, event_data, event_timestamp)
  ```

## 设备输出、指令相关函数

* on_output
  设备数据下置请求函数，如支持下置数据到设备。注册设备模型包含有下置数据的信息，然后在此函数实现设备数据下置。
  ```lua
  function app:on_output(src_app, sn, output, prop, value, timestamp)
  ```
* on_command
  如果应用创建的设备模型具备指令集，需要实现此函数来完成设备指令动作
  ```lua
  function app:on_command(src_app, sn, command, params)
  ```

## 应用控制交互函数

* on_ctrl
  应用之间可以进行一些非设备模型依赖的数据交换，行为控制。实现此函数可接收来自其他应用的请求，参考: ```app.api.send_ctrl```
  ```lua
  function app:on_ctrl(src_app, command, param)
  ```

## 设备输出、指令以及应用控制交互结果函数

* on_output_result
  如果应用请求了其他应用创建的设备模型中的数据下置，该数据下置被其他应用执行后，FreeIOE 会将结果传递给此函数
  ```lua
  function app:on_output_result(src_app, priv, result, err)
  ```
* on_command_result
  如果应用请求了其他应用创建的设备模型中的指令，该指令被其他应用执行后，FreeIOE 会将结果传递给此函数
  ```lua
  function app:on_command_result(src_app, priv, result, err)
  ```
* on_ctrl_result
  如果对其他应用发出了数据交换、行为控制，在对方执行完成后，FreeIOE 会回调此函数通知执行结果
  ```lua
  function app:on_ctrl_result(src_app, priv, result, err)
  ```
