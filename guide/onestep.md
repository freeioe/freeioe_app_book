
---

# 应用简要

本质上来说，FreeIOE应用是一个面向对象的Lua模块，并具备FreeIOE所规定的少量接口。

***下文皆以[middleclass](https://github.com/kikito/middleclass/wiki)为基础的示例。***


## 应用对象

应用对象必要接口：

1. 创建应用对象实体函数 
> initialize(name, sys, conf)
2. 应用对象启动函数
> start()
3. 应用对象推出函数
> close(reason)

应用模块可选接口：

1. 应用周期调用函数
> run(time_in_ms)


## 接口说明

* function app:initialize(name, sys, conf)
  * 参数说明
    * name: 对象实例名称
    * sys: 系统接口
    * conf: 应用实例配置信息
  * 用途
    * 本函数用以FreeIOE框架实例化应用对象，并传入应用初始化所需信息。
    * 应用不应在本函数实现复杂的逻辑。较为耗时和复杂的逻辑需要放到start函数中实现。


* function app:start()
  * 参数说明
    * 无
  * 用途
    * 设定应用回调接口
    * 创建设备对象实例
    * 设备通讯接口初始化
    * 其他应用所需初始化逻辑


* function app:close(reason)
  * 参数说明
    * reason: 退出的原因说明（文本）
  * 用途
    * 应用对象实例生命周期结束（非gc）的回调入口，用以应用清理对象等等。


* function app:run(tms)
  * 参数说明
    * tms：周期调用的间隔时间，单位毫秒。第一次调用值为1000毫秒，以后的值为本函数上一次运行的返回值。
  * 用途
    * 当应用有此接口，框架会周期调用此函数。周期间隔时间取决于本函数的返回值



