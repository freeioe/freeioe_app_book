
---

# 应用简要

本章介绍如何使用 **[Lua](http://www.lua.org/manual/5.3/)** 语言开发FreeIOE应用。

在开始阅读本章前，请先了解：

* 什么是Lua模块
* 如何构建简单的模块
* Lua中的面向对象编程

## APP组成

一个典型的FreeIOE 应用(APP)的结构如下：<br>
 ├── opcua\_client -------- ***App目录***<br>
 │----├── app.lua --------- ***App入口Lua文件***<br>
 │----├── conf.lua -------- ***App自定义模块文件***<br>
 │----└── luaclib --------- ***App自定义的C模块目录***<br>
 │----------└── opcua.so -- ***App自定义的OpcUA模块（C语言模块）***<br>

本质上来说，FreeIOE应用是一个面向对象的Lua模块，且实现了FreeIOE框架预定义的应用接口。

***下文皆以[middleclass](https://github.com/kikito/middleclass/wiki)为基础的示例。***

## 声明应用对象

``` lua
local class = require 'middleclass'
--- 注册对象(请尽量使用唯一的标识字符串)
local app = class('THIS_IS_AN_EXAMPLE_APP_FOR_FREEIOE')
--- 设定应用最小运行接口版本(目前版本为6,为了接口兼容性)
app.static.API_VER = 6
```

API_VER是标志当前应用运行的API需求版本号，请留意系统接口说明中的版本号信息。 当前FreeIOE最新版本号为6。如果无需兼容老版本的FreeIOE，请使用最新版本号


## 应用对象接口

FreeIOE预定义的应用接口：

1. 创建应用对象实体函数<br>
> initialize(name, sys, conf)
2. 应用对象启动函数<br>
> start()
3. 应用对象推出函数<br>
> close(reason)
4. 应用周期调用函数 (可选)<br>
> run(time_in_ms)

## 接口详细说明

* function app:initialize(name, sys, conf)
  * 参数说明
    * name: 对象实例名称
    * sys: 系统接口
    * conf: 应用实例配置信息
  * 返回值
    * 无
  * 用途
    * 本函数用以FreeIOE框架实例化应用对象，并传入应用初始化所需信息。
    * 应用不应在本函数实现复杂的逻辑。较为耗时和复杂的逻辑需要放到start函数中实现。


* function app:start()
  * 参数说明
    * 无
  * 返回值
    * boolean返回值，如果返回false/nil的话，框架则认为初始化失败，从而不再继续执行后续动作。
  * 用途
    * 设定应用回调接口
    * 创建设备对象实例
    * 设备通讯接口初始化
    * 其他应用所需初始化逻辑


* function app:close(reason)
  * 参数说明
    * reason: 退出的原因说明（文本）
  * 返回值
    * 无
  * 用途
    * 应用对象实例生命周期结束（非gc）的回调入口，用以应用清理对象等等。


* function app:run(tms)
  * 参数说明
    * tms：当前周期调用的间隔时间，单位毫秒。第一次调用值为1000毫秒，以后的值为本函数上一次运行的返回值。
  * 返回值
    * 返回下一次调用次函数的时间间隔（单位是毫秒)。
  * 用途
    * 当应用有此接口，框架会周期调用此函数。周期间隔时间取决于本函数的返回值



