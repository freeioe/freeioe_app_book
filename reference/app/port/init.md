
---

# 应用通讯端口模块

本模块封装了串口通讯模块初始化、TCP通讯模块初始化函数。帮助用户快速开发FreeIOE应用。

[使用参考](https://github.com/freeioe/freeioe_example_apps/blob/master/example/serial_socket/app.lua)

# 成员函数

## new_serial

创建串口通讯模块，此模块是封装了超时功能的serialchannel对象。

```lua
function _M.new_serial(conf, name)
end
```

## new_socket

创建网络通讯（TCP）模块，此模块是封装了超时功能的socketchannel对象

```lua
function _M.new_socket(conf, name)
end
```

## helper

app.port.helper模块

```lua
function _M.helper
end
```
