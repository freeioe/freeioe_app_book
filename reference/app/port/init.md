
---

# 应用通讯端口模块


本模块封装了串口通讯模块初始化、TCP通讯模块初始化函数。帮助用户快速开发FreeIOE应用。

[使用参考](https://github.com/freeioe/freeioe_example_apps/blob/master/example/serial_socket/app.lua)

## 模块函数

### new_serial
> function _M.new_serial(conf, name)

创建串口通讯模块，此模块是封装了超时功能的serialchannel对象。


### new_socket
> function _M.new_socket(conf, name)

创建网络通讯（TCP）模块，此模块是封装了超时功能的socketchannel对象

### helper
> function _M.helper

app.port.helper模块
