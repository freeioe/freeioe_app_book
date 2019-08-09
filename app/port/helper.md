
---

# 应用通讯端口帮助函数模块


本模块封装一些有助与使用通讯端口的函数。帮助用户快速开发FreeIOE应用。

[使用参考](https://github.com/freeioe/freeioe_example_apps/blob/master/example/serial_socket/app.lua)


## 模块函数

### read_socket
> function _M.read_socket(sock, len, dump)

读取套接字数据。

参数:
* sock --- socket对象 (socketchannel的request中response函数传入的socket对象)
* len --- 数据长度 
* dump --- 报文输出回调函数 function(string_data)

返回：
读取成功则读到的数据，否则返回nil

### read_serial
> function _M.read_serial(serial, len, dump, timeout)

读取串口数据。

参数:
* serial --- serial对象 (serialchannel的request中response函数传入的serial对象)
* len --- 数据长度 
* dump --- 报文输出回调函数 function(string_data)
* timeout --- 读取超时，单位是ms。默认是3000ms

返回：
读取成功则读到的数据，否则返回nil

