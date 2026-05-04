

---

# 应用工具函数模块

本模块提供FreeIOE应用的工具函数，包括实例名称验证和路径解析。

## valid_inst

验证应用实例名称是否有效

### 函数原型

```lua
function util.valid_inst(s)
end
```

### 参数说明

* s
  实例名称字符串

### 返回值

返回true表示有效，false表示无效

### 示例代码

```lua
local util = require 'app.util'
if util.valid_inst(app_name) then
    -- 实例名称有效
end
```

## dev_app_name

获取FreeIOE开发应用实例名称

### 函数原型

```lua
function util.dev_app_name()
end
```

### 返回值

返回开发应用名称 '_app'

## dev_app_path

获取FreeIOE开发应用路径

### 函数原型

```lua
function util.dev_app_path()
end
```

### 返回值

返回开发应用目录路径

## sys_app_name

获取FreeIOE系统应用实例名称

### 函数原型

```lua
function util.sys_app_name()
end
```

### 返回值

返回系统应用名称 'ioe'

## app_path

根据应用实例名称获取应用本地文件夹路径

### 函数原型

```lua
function util.app_path(app_inst)
end
```

### 参数说明

* app_inst
  应用实例名称

### 返回值

返回应用的本地文件夹路径
