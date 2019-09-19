
---

# 数据交叉计算工具模块 

> *** API_VER: 5 ***

本模块封装了数据交叉计算所需的基础代码和逻辑。帮助用户快速开发数据交叉计算应用

示例: [示例应用](https://github.com/freeioe/freeioe_example_apps/blob/master/computing/showbox/app.lua)


## 模块函数

### initialize
> function calc:initialize(sys, api, logger)

模块初始化：
* sys --- 系统接口
* api --- 应用接口
* logger --- 日志接口


### add
> function calc:add(name, inputs, trigger_cb, cycle_time)

增加交叉计算单元。

参数：

* name --- 单元名称
* inputs --- 设备数据输入项列表。例如： { {sn='device_sn', input='xxxx', prop='value, default=0}, {})
* trigger_cb --- 当设备数据输入项全部就绪或有变更时回调此函数，参数为数据当前值，顺序为inputs数组的顺序
* cycle_time --- 周期回调时间(如不指定，则不进行周期回调),单位是秒

### remove
> function calc:remove(name)

删除计算单元。

### start
> function calc:start(handler)

启动计算单元，handler是应用调用api:set_handler时使用的对象。

### stop
> function calc:stop()

停止计算单元。

