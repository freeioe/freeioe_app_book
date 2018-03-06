## Lua学习

Learning Lua: [http://lua-users.org/wiki/LearningLua](http://lua-users.org/wiki/LearningLua)  
 Sample Code: [http://lua-users.org/wiki/SampleCode](http://lua-users.org/wiki/SampleCode)  
 Module Tutorial: [http://lua-users.org/wiki/ModulesTutorial](http://lua-users.org/wiki/ModulesTutorial)

## APP组成

本章介绍如何使用Lua\(5.3\)语言开发IOT应用，在开始阅读本章前，请先熟悉Lua中模块的概念以及如何构建简单的Lua模块。  
 一个经典的APP会有如下的结构：  
 ├── opcua\_client — App目录  
 │   ├── app.lua — App入口Lua文件  
 │   ├── conf.lua — App自定义模块文件  
 │   └── luaclib — App自定义的C模块目录  
 │            └── opcua.so — App自定义的OpcUA模块（C语言模块）

APP应用的入口是一个符合IOT框架接口定义的特定Lua模块文件

## APP示例

```lua
 --- 导入需求的模块
 local class = require 'middleclass'
 local opcua = require 'opcua'
 --- 注册对象(请尽量使用唯一的标识字符串)
 local app = class("IOT_OPCUA_CLIENT_APP")
 --- 设定应用最小运行接口版本(目前版本为1,为了以后的接口兼容性)
 app.API_VER = 1
 ---
 -- 应用对象初始化函数
 -- @param name: 应用本地安装名称。 如modbus_com_1
 -- @param sys: 系统sys接口对象。参考API文档中的sys接口说明
 -- @param conf: 应用配置参数。由安装配置中的json数据转换出来的数据对象
 function app:initialize(name, sys, conf)
 self._name = name
 self._sys = sys
 self._conf = conf
 --- 获取数据接口
 self._api = sys:data_api()
 --- 获取日志接口
 self._log = sys:logger()
 self._connect_retry = 1000 end
 --- 应用启动函数
 function app:start()
 self._nodes = {}
 self._devs = {}
 --- 初始化OpcUA
 self._client = .....
 --- 设定接口处理接口
 self._api:set_handler({ on_output = function(...) print(...) end,
 on_ctrl = function(...) print(...) end })
 --- 创建设备对象
 local sys_id = self._sys:id()
 local dev = self._api:add_device(sys_id..'.OPCUA_TEST', inputs)
 return true end
 --- 应用退出函数
 function app:close(reason)
 print('close', self._name, reason)
 --- 断开opcua连接.. end
 --- 应用运行入口
 function app:run(tms)
 if not self._client then return 1000 end
 for sn, node in pairs(self._nodes) do local dev = self._devs[node.name]
 assert(dev)
 for k, v in pairs(node.vars) do --- 从OpcUa接口获取输入项当前值，并通过set_input_prop设定当前值
 dev:set_input_prop(k, "value", dv:asDouble(), now, 0) end end
 return 2000 end
 --- 返回应用对象
 return app
```



