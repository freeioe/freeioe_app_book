
---

# 设备通讯

常见的通讯方式有：

* TCP 套接字
* UDP 套接字
* 串口通讯

## TCP 套接字

FreeIOE框架提供TCP Socket连接有以下几种方式:

1. [SocketChannel](https://github.com/cloudwu/skynet/wiki/socketchannel) <br>
Skynet 框架提供的TCP Socket通讯框架。有两种工作模式:<br>
    * 同步模式
    * 异步模式(需要协议支持Session)
2. [app.socket](../app/socket.md)<br>
FreeIOE 封装的简易TCP Socket模式。
3. [Skynet Socket模块](https://github.com/cloudwu/skynet/wiki/socket)

参考示例应用库中的/modbus/master /modbus/slave /modbus/gateway 以及 /other/dtu 和 /example/serial_socket应用

## UDP 套接字

详见 Skynet Socket模块的说明。 并且FreeIOE扩展了Socket中的sendto函数，除了原本的sendto(id, from, data)之外，支持sendto(id, ip, port, data)方式直接指定发送目标的IP和端口信息。

## 串口通讯

FreeIOE 集成了 [librs232](http://github.com/srdgame/librs232) 模块，支持用户访问串口设备。

1. SerialChannel<br>
同SocketChannel模式的通讯框架
2. [app.serial](../app/serial.md)<br>
FreeIOE 封装的建议串口模块
3. [rs232](https://github.com/srdgame/librs232/blob/master/bindings/lua/rs232.lua)<br>
直接使用librs232模块

参考示例应用库中的modbus应用，以及 /other/dtu 和 /other/oliver_355_monitor、example/serial 等应用。