

---

# 循环缓冲区模块 (Cycle Buffer)

本模块实现了一个固定大小的循环缓冲区，当缓冲区满时自动丢弃最老的数据。

## 特性

- 固定大小缓冲区
- 自动丢弃最老数据
- 支持回调函数处理缓冲数据
- 提供批量处理能力

## 使用方法

```lua
local cyclebuffer = require 'buffer.cycle'

-- 创建循环缓冲区
-- size: 缓冲区最大容量
-- cb: 数据处理回调函数
local buf = cyclebuffer:new(function(...)
    print("Buffer data:", ...)
    return true  -- 返回true表示数据已处理，false表示重新入队
end, 100)

-- 推送数据到缓冲区
buf:push(data1, data2, data3)

-- 触发所有缓冲区数据
buf:fire_all()

-- 获取当前缓冲区大小
local size = buf:size()
```

## 接口说明

### new

创建循环缓冲区实例

#### 函数原型

```lua
function cyclebuffer:new(cb, size)
end
```

#### 参数说明

* cb
  数据处理回调函数，原型为 `function(...) end`
  - 返回true表示数据已处理，将从缓冲区移除
  - 返回false表示数据未处理，将保留在缓冲区
* size
  缓冲区最大容量

#### 返回值

返回循环缓冲区对象

### push

推送数据到缓冲区

#### 函数原型

```lua
function buffer:push(...)
end
```

#### 参数说明

* ...
  可变参数，要推送到缓冲区的数据

> 数据会先通过回调函数处理，如果回调返回false则存入缓冲区

### fire_all

触发所有缓冲区数据的处理

#### 函数原型

```lua
function buffer:fire_all(cb)
end
```

#### 参数说明

* cb
  可选的处理回调函数（默认使用创建时的回调）

#### 返回值

返回true表示所有数据已处理，false表示部分数据未处理

### size

获取当前缓冲区大小

#### 函数原型

```lua
function buffer:size()
end
```

#### 返回值

返回当前缓冲区中的数据数量
