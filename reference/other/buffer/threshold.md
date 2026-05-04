

---

# 阈值缓冲区模块 (Threshold Buffer)

本模块实现了一个带有时间窗口和计数限制的缓冲区，用于控制事件或数据的触发频率。

## 特性

- 时间窗口：只统计指定时间窗口内的数据
- 计数限制：当窗口内数据数量超过阈值时触发回调
- 自动清理过期数据
- 可选的丢弃回调

## 使用方法

```lua
local threshold_buffer = require 'buffer.threshold'

-- 创建阈值缓冲区
-- time_gap: 时间窗口（秒）
-- count: 触发阈值（事件数量）
-- callback: 达到阈值时的回调函数
-- on_drop_callback: 数据被丢弃时的回调函数（可选）
local buf = threshold_buffer:new(60, 20,
    function(...)
        print("Threshold reached:", ...)
    end,
    function(...)
        print("Data dropped:", ...)
    end
)

-- 推送数据
buf:push("event_data_1")
buf:push("event_data_2")

-- 获取当前计数
local count = buf:count()

-- 清理过期数据
buf:clean()
```

## 接口说明

### new

创建阈值缓冲区实例

#### 函数原型

```lua
function threshold_buffer:new(time_gap, count, callback, on_drop_callback)
end
```

#### 参数说明

* time_gap
  时间窗口，单位秒
* count
  触发阈值，在时间窗口内达到此数量时触发回调
* callback
  触发回调函数
* on_drop_callback
  可选的数据丢弃回调函数

#### 返回值

返回阈值缓冲区对象

### push

推送数据到缓冲区

#### 函数原型

```lua
function buffer:push(...)
end
```

#### 参数说明

* ...
  可变参数，要推送的数据

#### 返回值

成功返回回调函数的返回值，超过阈值时返回nil和错误信息

### clean

清理过期的数据

#### 函数原型

```lua
function buffer:clean()
end
```

> 此方法会在每次push时自动调用，通常不需要手动调用

### count

获取当前窗口内的数据计数

#### 函数原型

```lua
function buffer:count()
end
```

#### 返回值

返回当前时间窗口内的数据数量
