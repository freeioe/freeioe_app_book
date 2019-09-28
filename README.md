
---

# FreeIOE应用开发指南

FreeIOE 应用开发指南


## 基础介绍

FreeIOE基础介绍

* [FreeIOE是什么](intro/freeioe.md)
* [名词解释](intro/glossary.md)
* [什么是物联网](intro/iot.md)


## 开发环境搭建

搭建开发环境

* [注册开发者账户](dev_setup/cloud_reg.md)
* [虚拟机开发环境](dev_setup/vbox.md)
* [Linux原生开发环境](dev_setup/linux.md)
* [应用中心在线开发](dev_setup/app_center.md)
* [设备Web在线开发](dev_setup/dev_web.md)
* [使用VSCode插件开发](dev_setup/vscode-extension.md)


## 开发引导

了解开发应用的细节

* [应用简要](guide/onestep.md)
* [开发指南](guide/tutorial.md)
* [应用示例](guide/example.md)
* [应用示例代码库 1](https://github.com/freeioe/freeioe_example_apps)
* [应用示例代码库 2](https://github.com/viccom/myfreeioe_apps)


## 应用发布

发布您的应用到应用中心，以便在设备中使用它

* [申请账户](app_center/reg.md)
* [应用打包](app_center/pack.md)
* [版本上传](app_center/upload.md)


## 应用接口文档

FreeIOE给应用开发提供的接口列表

* [系统接口](app/sys.md)
* [基础接口](app/api.md)
* [设备对象](app/device.md)
* [统计接口](app/stat.md)
* [日志接口](app/logger.md)
* [事件类型和等级](app/event.md)
* [云配置接口](app/conf_api.md)
* [云配置帮助接口](app/conf_helper.md)
* [基础应用封装模块](app/base/init.md)
* [MQTT应用封装模块](app/base/mqtt.md)
* 其他模块
	* [数据通讯模块](app/port/README.md)
	* [工具类模块](app/utils/README.md)

## 其它资料 & 文档

其它有关的资料索引

* [Lua语言学习](other/learning_lua.md)
* [二进制数据操作](other/binary.md)
* [文件操作](other/file.md)
* [模块列表](other/modules.md)
* 模块说明
    * [IOE模块](other/ioe.md)
    * [串口操作模块](other/serialdriver.md)
    * [累计量计算模块](other/summation.md)
* 工具模块
    * [helper模块](other/utils/helper.md)
    * [leds模块](other/utils/leds.md)
    * [gpios模块](other/utils/gpios.md)
    * [sysinfo模块](other/utils/sysinfo.md)

