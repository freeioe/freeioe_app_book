
---

# 统计接口

用于设备采集的通讯状态统计。

## 统计属性列表

统计数据属性是一个受限的集合，列表如下

* status
  当前链路状态
* success_ratio
  成功率
* error_ratio
  错误率
* packets_in
  接收包数
* packets_out
  发送包数
* packets_error
  错误包数(一般指接收错误包)
* bytes_in
  接受字节数
* bytes_out
  发送字节数
* bytes_error
  错误字节数（一般指接受到的错误字节数)

**注：**
一个设备（网关设备或其他采集设备)都可以拥有多个统计集合，参考device:stat接口

## get

获取统计属性当前值

### 函数原型

```lua
function stat:get(prop)
end
```

## reset

重置统计属性值

### 函数原型

```lua
function stat:reset(prop)
end
```

## inc

累加统计属性当前值

### 函数原型

```lua
function stat:inc(prop, value)
end
```

## set

### 函数原型

设定统计属性值

```lua
function stat:set(prop, value)
end
```

### cleanup

接口清理接口(内部使用)

```lua
function stat:cleanup()
end
```