
---

# 统计接口

用于设备采集的通讯状态统计。

## 统计属性列表

统计数据属性是一个受限的集合，列表如下

* status <br>当前链路状态
* success_ratio <br>成功率
* error_ratio <br>错误率
* packets_in <br> 接收包数
* packets_out <br> 发送包数
* packets_error <br> 错误包数(一般指接收错误包)
* bytes_in <br> 接受字节数
* bytes_out <br> 发送字节数
* bytes_error <br> 错误字节数（一般指接受到的错误字节数)

注：一个设备（网关设备或其他采集设备)都可以拥有多个统计集合，参考device:stat接口


### get
> function stat:get(prop)

获取统计属性当前值


### reset
> function stat:reset(prop)

重置统计属性值


### inc
> function stat:inc(prop, value)

累加统计属性当前值


### set
> function stat:set(prop, value)

设定统计属性值


### cleanup
> function stat:cleanup()

接口清理接口(内部使用)
