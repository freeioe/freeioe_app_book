
---

# 数据订阅计算工具模块

> *** API_VER: 5 ***

本模块封装了数据订阅计算所需的基础代码和逻辑，帮助用户快速开发数据交叉计算应用

1. 确保进行计算时，数据点的数据是就绪的
2. 确保当数据变化时，才进行计算
3. 支持周期性回调

## initialize

模块实例构造函数

#### 函数原型

```lua
function calc:initialize(sys, api, logger)
```

#### 参数

* sys
   系统接口
* api
   应用接口
* logger
   日志接口

#### 示例代码

```lua
local app_calc = require 'app.utils.calc'

local calc = app_calc:new(self._sys, self._api, self._logger)
```

## add

增加交叉计算单元

#### 函数原型

```lua
function calc:add(name, inputs, trigger_cb, cycle_time)
```

#### 参数

* name
  单元名称
* inputs
  设备数据输入项列表。
* trigger_cb
  当设备数据输入项全部就绪或有变更时回调此函数，参数为数据当前值，顺序为inputs数组的顺序
* cycle_time
  周期回调时间(如不指定，则不进行周期回调),单位是秒

#### 示例代码

```lua

local inputs = {
    {sn='device_sn', input='Va', prop='value', default=0},
    {sn='device_sn', input='Vb', prop='value', default=0}
}

calc:add('example_calc_unit', inputs, function(Va, Vb)
    --- Your code here
end)
```

## remove

删除交叉计算单元

#### 函数原型

```lua
function calc:remove(name)
```

删除计算单元。

#### 参数

* name
  单元名称

#### 示例代码

```lua
calc:remove('example_calc_unit')
```

## start

添加计算单元后，应该调用此方法，让模块背后的逻辑启动运行。

#### 函数原型

```lua
function calc:start(handler)
```

启动计算单元，handler是应用调用api:set_handler时使用的对象。

## stop

停止计算单元后，所有添加的计算单元将不再被执行。

#### 函数原型

```lua
function calc:stop()
```

停止计算单元。
