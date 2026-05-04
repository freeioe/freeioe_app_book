

---

# 变化上报模块 (COV)

本模块提供数据变化检测功能，仅在数据发生变化或超过时间阈值时才触发回调，用于减少不必要的数据传输和处理。

## 特性

- 支持数字、字符串、表类型数据的变化检测
- 可配置浮点数变化阈值
- 支持周期性强制上报（TTL - Time To Live）
- 支持批量数据处理
- 可配置的最小TTL检测间隔

## 使用方法

```lua
local cov = require 'cov'

-- 创建COV实例
local cov_obj = cov:new(function(key, value, timestamp, quality)
    -- 数据变化时的回调处理
    print('Data changed:', key, value)
end, {
    float_threshold = 0.001,  -- 浮点数变化阈值（默认0.000001）
    ttl = 60,                -- 最长60秒强制上报一次
    min_ttl_gap = 10         -- TTL检测间隔，单位1/10秒（默认10）
})

-- 处理单个数据变化
cov_obj:handle('device/input/value', 10.5, ioe.time(), 0)

-- 批量处理数据
local datas = {
    {'device1', 'temp', 'value', 25.3, ioe.time(), 0},
    {'device1', 'humidity', 'value', 60.2, ioe.time(), 0}
}
local changed = cov_obj:handle_batch(datas)

-- 启动TTL定时器
cov_obj:start()

-- 停止定时器
cov_obj:stop()

-- 获取当前快照
local snapshot = cov_obj:snapshot()

-- 清空缓存
cov_obj:clean()
```

## 接口说明

### new

创建COV实例

#### 函数原型

```lua
function cov:new(cb, opt)
end
```

#### 参数说明

* cb
  回调函数，原型为 `function(key, value, timestamp, quality) end`
* opt
  配置选项表

#### 配置选项

* float_threshold
  浮点数变化阈值（默认 0.000001）
* ttl
  最长上报时间间隔，单位秒（可选）
* min_ttl_gap
  TTL检测间隔，单位1/10秒（默认10）
* try_convert_string
  是否尝试将字符串转换为数字（默认false）
* disable
  是否禁用变化检测（默认false）

#### 返回值

返回COV对象

### handle

处理单个数据变化检测

#### 函数原型

```lua
function cov:handle(key, value, timestamp, quality)
end
```

#### 参数说明

* key
  数据键，用于标识数据点
* value
  数据值（支持number、string、table类型）
* timestamp
  时间戳
* quality
  质量标识（0表示良好）

#### 返回值

返回true表示未触发回调（数据未变化），返回nil或false表示已触发回调

### handle_batch

批量处理数据变化检测

#### 函数原型

```lua
function cov:handle_batch(datas, fire_cb, key_cb)
end
```

#### 参数说明

* datas
  数据数组，每个元素格式为 `{key, input, prop, value, timestamp, quality}`
* fire_cb
  可选的触发回调函数（默认使用创建时的回调）
* key_cb
  可选的键提取函数，用于从数据项中提取键和值位置

#### 返回值

返回发生变化的数据数组

### fire_snapshot

触发当前缓存的所有数据快照

#### 函数原型

```lua
function cov:fire_snapshot(cb)
end
```

#### 参数说明

* cb
  可选的回调函数（默认使用创建时的回调）

### snapshot

获取当前缓存的数据快照

#### 函数原型

```lua
function cov:snapshot()
end
```

#### 返回值

返回以键为索引的数据表

### clean

清空缓存

#### 函数原型

```lua
function cov:clean()
end
```

### clean_with_match

根据匹配条件清理缓存

#### 函数原型

```lua
function cov:clean_with_match(mfunc)
end
```

#### 参数说明

* mfunc
  匹配函数，原型为 `function(key) return true/false end`，返回true表示删除该键

### start

启动TTL定时器

#### 函数原型

```lua
function cov:start()
end
```

> 只有在设置了ttl选项时才有效

### stop

停止TTL定时器

#### 函数原型

```lua
function cov:stop()
end
```

### timer

手动触发定时器检查（通常不需要手动调用）

#### 函数原型

```lua
function cov:timer(now, cb)
end
```

#### 参数说明

* now
  当前时间（ioe.time()返回的秒数）
* cb
  回调函数

#### 返回值

返回下次检查的时间间隔
