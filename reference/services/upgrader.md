
---

# 软件分发服务

FreeIOE 提供的软件分发服务，可以管理应用和系统的分发。

## 安装应用

通过这个接口，可以让 FreeIOE 从平台安装指定的应用。

```lua
local args = {
    name='APP00000040',
    inst='modbus_1',
    version=36,
    conf={
        port='/dev/ttyS1'
    }
}
local action_id = 'this_is_an_example_unique_id'
local r, err = skynet.call(".upgrader", "lua", "install_app", action_id, args)
```

## 升级应用

通过这个接口，可以让 FreeIOE 从平台安装指定的应用的新版本。

```lua
local args = {
    name='APP00000040',
    inst='modbus_1',
    version=39
}
local action_id = 'this_is_an_example_unique_id'
local r, err = skynet.call(".upgrader", "lua", "upgrade_app", action_id, args)
```

## 应用更名

通过这个接口，可以更改应用实例名。

```lua
local args = {
    inst='modbus_1',
    new_name='modbus_2'
}
local action_id = 'this_is_an_example_unique_id'
local r, err = skynet.call(".upgrader", "lua", "rename_app", action_id, args)
```

## 删除应用

通过这个接口，可以更改应用实例名。

```lua
local args = {
    inst='modbus_1'
}
local action_id = 'this_is_an_example_unique_id'
local r, err = skynet.call(".upgrader", "lua", "uninstall_app", action_id, args)
```

## 枚举应用列表

通过这个接口，可以获取应用列表，以及应用的信息。

```lua
local apps = skynet.call(".upgrader", "lua", "list_app")
```

## 升级 FreeIOE

```lua
local args = {
    no_ack = 1,
    version = 1192,
    skynet = {
        version = 1934
    }
},
local action_id = 'this_is_an_example_unique_id'
local r, err = skynet.call(".upgrader", "lua", "uninstall_app", action_id, args)
```

