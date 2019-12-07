
---

# 应用管理服务

FreeIOE 提供的应用管理服务。

## 获取应用列表

通过这个接口，可以启动已经停止的应用。

```lua
local appmgr = snax.queryservice('appmgr')
local apps = appmgr.req.list()
```

## 启动应用

通过这个接口，可以启动已经停止的应用。

```lua
local appmgr = snax.queryservice('appmgr')
local r, err = appmgr.req.start('modbus_1', conf)
```

## 重启应用

通过这个接口，可以重启正在运行的应用。

```lua
local appmgr = snax.queryservice('appmgr')
local r, err = appmgr.req.restart('modbus_1', 'restart app from user')
```

## 获取应用配置

通过这个接口，可以更改应用实例当前的配置信息。

```lua
local appmgr = snax.queryservice('appmgr')
local conf, err = appmgr.req.get_conf('modbus_1')
```

## 更改应用配置

通过这个接口，可以更改应用实例当前的配置信息。

```lua
local appmgr = snax.queryservice('appmgr')
local r, err = appmgr.req.set_conf('modbus_1', conf)
```

## 更改应用选项

通过这个接口，可以修改应用的选项。如自动启动等

```lua
local appmgr = snax.queryservice('appmgr')
local r, err = appmgr.req.app_option('modbus_1', 'auto', 1)
```

## 应用改名

通过这个接口，可以修改应用实例名称

```lua
local appmgr = snax.queryservice('appmgr')
local r, err = appmgr.req.app_rename('modbus_1', 'modbus_2', "change the name by cloud")
```

## 停止所有应用

通过这个接口触发停止所有应用的请求。

```lua
local appmgr = snax.queryservice('appmgr')
appmgr.post.close_all("close all applications by cloud")
```
