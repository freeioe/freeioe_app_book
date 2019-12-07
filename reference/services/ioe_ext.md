
---

# 扩展管理服务

FreeIOE 提供的扩展管理服务，可以管理应用所使用的 FreeIOE 扩展。

## 枚举扩展列表

获取当前 FreeIOE 已经安装的扩展列表信息。

```lua
local list, err = skynet.call(".ioe_ext", "lua", "list")
```

## 升级扩展

升级已经安装的扩展的版本

```lua
local args = {
    name='opcua',
    version=39
}
local action_id = 'this_is_an_example_unique_id'
local r, err = skynet.call(".ioe_ext", "lua", "upgrade_ext", action_id, args)
```

## 自动清理

自动清理已经不再被使用的 FreeIOE 扩展包

```lua
local r, err = skynet.call(".ioe_ext", "lua", "auto_clean")
```
