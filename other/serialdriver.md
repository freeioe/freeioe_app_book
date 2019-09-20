
----

# 串口读写模块

对librs232模块进行的实用性封装。


### new

> function serial:new(port, baudrate, data_bits, parity, stop_bits, flowcontrol)

初始化模块对象。包含以下参数：

| --- | --- | --- |
| port | string | 串口路径（windows下是串口名称,如COM1， Linux下是路径，如/dev/ttyS1）|
| baudrate | number | 波特率，如9600, 115200等 |
| data_bits | number | 数据位长度 7/8/9 |
| parity | string | 校验模式ODD/EVEN/NONE |
| stop_bits | number | 停止位 1/2 |
| flowcontrol | string | 流控 ON/OFF |


### open

> funciton serial:open()

打开串口，失败返回nil和错误信息


### close

> function serial:close()

关闭串口


### write

> function serial:write(data)

发送串口数据。失败返回nil和错误信息


### flush

> function serial:flush()

发送系统缓存的串口数据。


### in_queue_clear

> function serial:in_queue_clear()

清空接收缓存数据


### in_queue

> function serial:in_queue()

获取当前接收缓存长度


### device

> function serial:device()

获取当前串口路径或名称


### fd

> function serial:fd()

获取文件句柄


### set_baud_rate

> function serial:set_baud_rate(baudrate)

设置串口波特率


### baud_rate

> function serial:baud_rate()

获取当前波特率


### set_data_bits

> function serial:set_data_bits(data_bits)

设定数据位长度


### data_bits

> function serial:data_bits()

获取数据位长度


### set_parity

> function serial:set_parity(parity)

设定校验模式


### parity

> function serial:parity()

获取当前校验模式


### set_flow_control

> function serial:set_flow_control(flowcontrol)

设定流控方式


### flow_control

> function serial:flow_control()

获取当前流控方式


### set_dtr

> function serial:set_dtr(dtr)


### dtr

> function serial:dtr()


### set_rts

> function serial:set_rts(rts)


### rts

> function serial:rts()


### start

> function serial:start(callback, timeout)

开启串口读取协程：

* callback <br> 数据回调 function(data, err) 如有串口错误，data为nil
* timeout <br> 读取超时时间，单位为ms
