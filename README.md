
---

# FreeIOE应用开发指南

FreeIOE 应用开发指南

## 基础介绍

FreeIOE基础介绍

* [什么是 FreeIOE](intro/freeioe.md)
* [名词解释](intro/glossary.md)


## 开发引导

了解开发应用的细节

* [开发入门](guide/intro.md)
* [网关管理维护](guide/using_gateway.md)

### 开发环境搭建

搭建开发环境

* [注册开发者账户](guide/dev_setup/cloud_reg.md)
* [虚拟机开发环境](guide/dev_setup/vbox.md)
* [使用VSCode插件开发](guide/dev_setup/vscode-extension.md)
* [Linux原生开发环境](guide/dev_setup/linux.md)
* [应用中心在线开发](guide/dev_setup/app_center.md)
* [设备Web在线开发](guide/dev_setup/dev_web.md)

### 应用开发

* [从零构造应用](guide/app_dev/from_zero.md)
* [完整应用示例](guide/app_dev/example.md)
* [快速构建应用](guide/app_dev/with_base.md)
* [注意事项](guide/app_dev/tips.md)

### 应用示例集

* [应用示例代码库 1](https://github.com/freeioe/freeioe_example_apps)
* [应用示例代码库 2](https://github.com/viccom/myfreeioe_apps)

### 应用发布

发布您的应用到应用中心，以便在设备中使用它

* [申请账户](app_center/reg.md)
* [应用打包](app_center/pack.md)
* [版本上传](app_center/upload.md)

## 接口文档

FreeIOE给应用开发提供的接口列表

### 应用接口

* [系统接口](reference/app/sys.md)
* [基础接口](reference/app/api.md)
* [设备对象](reference/app/device.md)
* [统计接口](reference/app/stat.md)
* [日志接口](reference/app/logger.md)
* [事件类型和等级](reference/app/event.md)
* [云配置接口](reference/app/conf_api.md)
* [云配置帮助接口](reference/app/conf_helper.md)
* [基础应用封装模块](reference/app/base/init.md)
* [MQTT应用封装模块](reference/app/base/mqtt.md)
* 其他模块
  * [数据通讯模块](reference/app/port/README.md)
  * [工具类模块](reference/app/utils/README.md)

### 其它资料 & 文档

其它有关的资料索引

* [Lua语言学习](reference/other/learning_lua.md)
* [二进制数据操作](reference/other/binary.md)
* [文件操作](reference/other/file.md)
* [模块列表](reference/other/modules.md)
* 模块说明
  * [IOE模块](reference/other/ioe.md)
  * [串口操作模块](reference/other/serialdriver.md)
  * [累计量计算模块](reference/other/summation.md)
* 工具模块
  * [helper模块](reference/other/utils/helper.md)
  * [leds模块](reference/other/utils/leds.md)
  * [gpios模块](reference/other/utils/gpios.md)
  * [sysinfo模块](reference/other/utils/sysinfo.md)
  * [周期计时模块](reference/other/utils/timer.md)

