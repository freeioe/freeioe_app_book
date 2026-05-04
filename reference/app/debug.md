

---

# 调试工具模块

本模块提供应用的调试工具，包括缓冲区打包用于诊断目的。

## initialize

初始化调试工具实例

### 函数原型

```lua
function debug:initialize(app_name, logger)
end
```

### 参数说明

* app_name
  应用名称
* logger
  可选的日志实例（默认创建新实例）

## pack_all

将应用的所有通信和日志数据打包到一个文件中

查询缓冲区服务获取应用特定数据并保存为JSON文件

### 函数原型

```lua
function debug:pack_all()
end
```

### 返回值

成功返回文件名，失败返回nil和错误消息

### 示例代码

```lua
local debug = require 'app.debug'
local dbg = debug:new('my_app')
local filename, err = dbg:pack_all()
if filename then
    print('Debug data saved to:', filename)
else
    print('Error:', err)
end
```
