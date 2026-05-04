

---

# 云配置项接口

云服务提供配置项版本发布服务。类似应用版本一样，支持用户发布版本用于更新配置项。云配置项是与云应用关联，并由用户自定义并发布的配置项文件。

* 文件包括

	* 应用配置
	* 设备模板
	* 其他配置。


* FreeIOE还提供基于设备模板和设备列表的上层[配置帮助接口](conf_helper.md)。


## initialize

构造函数

### 函数原型

```lua
function api:initialize(sys, app, conf, ext, dir)
end
```

### 参数说明

* sys
  系统API对象
* app
  应用ID。e.g. APP00000001
* conf
  应用云配置ID。 e.g. TPL000000001 CNF000000001
* ext
  配置文件本地存储的扩展名。默认为csv
* dir
  配置文件本地存储的子目录名。 默认为tpl

### 示例代码

``` lua
local conf_api = require 'app.conf.api'
local api = conf_api:new(sys, 'APP0000001', self._config)
```

## version

获取当前配置的最新版本号

### 函数原型

```lua
function api:version()
end
```

### 返回值

返回最新版本号

## data

获取指定版本的配置项内容。

### 函数原型

```lua
function api:data(version)
end
```

### 参数说明

* version
  配置版本号（nil表示获取最新版本）

### 返回值

成功返回配置数据字符串和版本号，失败返回nil和错误消息

> 注，如本机已经获取过指定版本，会读取本地缓存的文件

## fetch

获取配置并返回本地文件路径

### 函数原型

```lua
function api:fetch(version)
end
```

### 参数说明

* version
  配置版本号（nil表示获取最新版本）

### 返回值

成功返回本地文件路径，失败返回nil和错误消息
