

---

# 应用通讯端口模块

本模块封装了串口通讯模块初始化、TCP通讯模块初始化函数。帮助用户快速开发FreeIOE应用。

[使用参考](https://github.com/freeioe/freeioe_example_apps/blob/master/example/serial_socket/app.lua)

# 成员函数

## new_serial

创建串口通讯模块，此模块是封装了超时功能的serialchannel对象。

### 函数原型

```lua
function port.new_serial(conf, name)
end
```

### 参数说明

* conf
  串口配置表
* name
  串口名称

### 返回值

返回timeout_channel包装的串口通道对象

## new_socket

创建网络通讯（TCP）模块，此模块是封装了超时功能的socketchannel对象

### 函数原型

```lua
function port.new_socket(conf, name)
end
```

### 参数说明

* conf
  网络配置表
* name
  连接名称

### 返回值

返回timeout_channel包装的socket通道对象

## new_agent_serial

创建串口代理模块，支持在多个应用实例之间共享串口

> *** API_VER: 4 ***

### 函数原型

```lua
function port.new_agent_serial(opt, shared_name)
end
```

### 参数说明

* opt
  选项配置表
* shared_name
  共享名称字符串，用于在应用实例之间共享此串口

### 返回值

返回agent_serial对象

## new_agent_socket

创建网络代理模块，支持在多个应用实例之间共享socket连接

> *** API_VER: 4 ***

### 函数原型

```lua
function port.new_agent_socket(opt, shared_name)
end
```

### 参数说明

* opt
  选项配置表
* shared_name
  共享名称字符串，用于在应用实例之间共享此socket

### 返回值

返回agent_socket对象

## helper

app.port.helper模块

### 函数原型

```lua
function port.helper
end
```

### 返回值

返回helper模块
