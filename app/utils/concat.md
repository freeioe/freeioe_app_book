
---

# 数据拼接工具模块 

> *** API_VER: 5 ***

本模块封装了拼接多个数据逻辑。帮助用户快速开发数据拼接计算应用

示例: [示例应用](https://github.com/freeioe/freeioe_example_apps/blob/master/opcua/yizumi)


## 模块函数

### initialize
> function concat:initialize(func, need_all, delay, timeout)

模块初始化：
* func --- 数据拼接回调
* need_all --- 是否需要所有数据就绪才调用拼接函数
* delay --- 单个数据更新后等待其他数据就绪的时间(单位是毫秒ms)
* timeout --- 数据项超时无效的时长(单位是秒)

### add
> function concat:add(key, default, delay, timeout)

增加数据项。

参数：

* key --- 数据关键字
* default --- 默认数值、或者数据超时无效后参与计算的数值
* delay --- 发起计算的等待其他数据的时长，默认使用初始化中的delay时长
* timeout --- 数据项无效时长，默认使用初始化中的timeout时常

### update
> function concat:update(key, value, timestamp, quality)

更新单个数据。

参数:

* key --- 数据关键字
* value --- 数据值
* timestamp --- 数据时间戳
* quality --- 质量戳

### call_calc
> function concat:call_calc()

触发计算回调，一般使用情况下，无需主动使用此函数。 


