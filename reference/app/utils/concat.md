
---

# 数据拼接

> *** API_VER: 5 ***

本模块封装了拼接多个数据的逻辑，让用户可以轻松处理：

1. 来自多个数据点的数据拼接成为一个数据
2. 处理数据拼接的时间有效性要求

示例: [示例应用](https://github.com/freeioe/freeioe_example_apps/blob/master/opcua/yizumi)


## initialize

#### 函数原型

```lua
function concat:initialize(func, need_all, delay, timeout)
```

#### 参数说明

* func
  数据拼接回调
* need_all
  是否需要所有数据就绪才调用拼接函数
* delay
  单个数据更新后等待其他数据就绪的时间(单位是毫秒ms)
* timeout
  数据项超时无效的时长(单位是秒)

#### 使用示例

```lua
local app_concat = require 'app.utils.concat'

local concat = app_concat:new(function(values)
end, true, 200)
```

## add

增加数据项

#### 函数原型

```lua
function concat:add(key, default, delay, timeout)
```

#### 参数说明

* key
  数据关键字，可以是数字或字符串
* default
  默认数值、或者数据超时无效后参与计算的数值
* delay
  发起计算的等待其他数据的时长，默认使用初始化中的delay时长
* timeout
  数据项无效时长，默认使用初始化中的timeout时常

#### 示例代码

```lua
concat:add('temp1')
concat:add('temp2')
```

## update

更新单个数据项的数据

#### 函数原型

```lua
function concat:update(key, value, timestamp, quality)
```

#### 参数说明

* key
  数据关键字
* value
  数据值
* timestamp
  数据时间戳
* quality
  质量戳

## 主动触发一次回调

触发计算回调，一般使用情况下，无需主动使用此函数。

#### 函数原型

```lua
function concat:call_calc()
```
