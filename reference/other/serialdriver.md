
----

# 串口读写模块

对librs232模块进行的实用性封装。

## new

初始化模块对象。

```lua
function serial:new(port, baudrate, data_bits, parity, stop_bits, flowcontrol)
end
```

### 参数说明

| --- | --- | --- |
| port | string | 串口路径（windows下是串口名称,如COM1， Linux下是路径，如/dev/ttyS1）|
| baudrate | number | 波特率，如9600, 115200等 |
| data_bits | number | 数据位长度 7/8/9 |
| parity | string | 校验模式ODD/EVEN/NONE |
| stop_bits | number | 停止位 1/2 |
| flowcontrol | string | 流控 ON/OFF |

## open

打开串口，失败返回nil和错误信息

```lua
funciton serial:open()
end
```

## close

关闭串口

```lua
function serial:close()
end
```

## write

发送串口数据。失败返回nil和错误信息

```lua
function serial:write(data)
end
```

## flush

发送系统缓存的串口数据。

```lua
function serial:flush()
end
```

## in_queue_clear

清空接收缓存数据

```lua
function serial:in_queue_clear()
end
```

## in_queue

获取当前接收缓存长度

```lua
function serial:in_queue()
end
```

## device

获取当前串口路径或名称

```lua
function serial:device()
end
```

## fd

获取文件句柄

```lua
function serial:fd()
end
```

## set_baud_rate

设置串口波特率

```lua
function serial:set_baud_rate(baudrate)
end
```

## baud_rate

获取当前波特率

```lua
function serial:baud_rate()
end
```

## set_data_bits

设定数据位长度

```lua
function serial:set_data_bits(data_bits)
end
```

## data_bits

获取数据位长度

```lua
function serial:data_bits()
end
```

## set_parity

设定校验模式

```lua
function serial:set_parity(parity)
end
```

## parity

获取当前校验模式

```lua
function serial:parity()
end
```

## set_flow_control

设定流控方式

```lua
function serial:set_flow_control(flowcontrol)
end
```

## flow_control

获取流控方式

```lua
function serial:flow_control()
end
```

获取当前流控方式

## set_dtr

设置 dtr 参数

```lua
function serial:set_dtr(dtr)
end
```

## dtr

获取 dtr 参数

```lua
function serial:dtr()
end
```

## set_rts

设置 rts 参数

```lua
function serial:set_rts(rts)
end
```

## rts

获取 rts 参数

```lua
function serial:rts()
end
```

## start

开启串口读取协程：

```lua
function serial:start(callback, timeout)
end
```

### 参数说明

* callback
  数据回调 function(data, err) 如有串口错误，data为nil
* timeout
  读取超时时间，单位为ms
