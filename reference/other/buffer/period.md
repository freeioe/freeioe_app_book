

---

# 批次数据缓冲区模块 (Period Buffer)

本模块实现了一个周期性批量处理数据的缓冲区，支持定时批量发送和数据丢弃控制。

## 特性

- 周期性批量处理数据
- 可配置的最大缓冲区大小
- 支持最大批次大小限制
- 自动数据丢弃控制（当缓冲区满时）
- 支持数据丢弃回调

## 使用方法

```lua
local periodbuffer = require 'buffer.period'

-- 创建批次缓冲区
-- period: 处理周期，单位毫秒
-- max_buf_size: 最大缓冲区大小
-- max_batch_size: 最大批次大小（可选，默认1024）
local buf = periodbuffer:new(1000, 1000, 100)

-- 设置处理和丢弃回调
buf:start(function(data)
    -- 处理批次数据
    for _, item in ipairs(data) do
        print("Process:", item)
    end
    return true
end,
    function(dropped_data)
    -- 处理被丢弃的数据
    print("Dropped:", dropped_data)
end)

-- 推送数据到缓冲区
buf:push(1, 2, 3)
buf:push(4, 5, 6)

-- 触发所有数据处理
buf:fire_all()

-- 获取当前缓冲区大小
local size = buf:size()

-- 停止周期处理
buf:stop()
```

## 接口说明

### new

创建批次缓冲区实例

#### 函数原型

```lua
function periodbuffer:new(period, max_buf_size, max_batch_size)
end
```

#### 参数说明

* period
  处理周期，单位毫秒
* max_buf_size
  最大缓冲区大小
* max_batch_size
  最大批次大小（可选，默认1024）

#### 返回值

返回批次缓冲区对象

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

> 当缓冲区满时，最老的数据会被丢弃

### fire_all

触发所有缓冲区数据的处理

#### 函数原型

```lua
function buffer:fire_all(cb)
end
```

#### 参数说明

* cb
  可选的处理回调函数（默认使用start时设置的回调）

#### 返回值

返回true表示所有数据处理成功，false表示部分数据未处理成功

### size

获取当前缓冲区大小

#### 函数原型

```lua
function buffer:size()
end
```

#### 返回值

返回当前缓冲区中的数据数量

### reinit

重新初始化缓冲区参数

#### 函数原型

```lua
function buffer:reinit(period, size)
end
```

#### 参数说明

* period
  新的处理周期（可选，默认保持原值）
* size
  新的最大缓冲区大小（可选，默认保持原值）

### period

获取当前处理周期

#### 函数原型

```lua
function buffer:period()
end
```

#### 返回值

返回处理周期（毫秒）

### max_size

获取最大缓冲区大小

#### 函数原型

```lua
function buffer:max_size()
end
```

#### 返回值

返回最大缓冲区大小

### start

启动周期性处理

#### 函数原型

```lua
function buffer:start(cb, drop_cb)
end
```

#### 参数说明

* cb
  数据处理回调函数，原型为 `function(data) end`
  - data: 数据数组，每个元素是原始推送的数据包
* drop_cb
  可选的数据丢弃回调函数，原型为 `function(...) end`

> 启动后会自动开始周期性处理

### stop

停止周期性处理

#### 函数原型

```lua
function buffer:stop()
end
```
