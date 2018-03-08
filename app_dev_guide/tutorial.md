
---

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
 │        └── opcua.so — App自定义的OpcUA模块（C语言模块）

APP应用的入口是一个符合IOT框架接口定义的特定Lua模块文件

## 开发环境

* 本地开发环境
  * 使用虚拟机开发
  * Linux原生开发环境
* 设备在线开发
  * 应用中心在线开发
  * 设备web在线开发



## 



