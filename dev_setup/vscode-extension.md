---

# 使用VS Code开发FreeIOE应用


## 下载VS Code

可以从微软网站下载VS Code。[地址](https://code.visualstudio.com/)


## 安装IOT Editor插件

运行VS Code后，安装IOT Editor插件。

> 另：建议安装Lua语言插件:
> * luaide-lite


## 使用 IOT Editor插件

* 打开工作目录
* 保存工作区
* Shift + Alt + P 搜索运行 "IOT: Setup editor workspace"
* 在打开的freeioe_devices.json文件中的devices节点下，输入正确的设备IP，并修改其设备名称
* Shift + Alt + P 搜索运行 "IOT: Connect to freeioe device"
* 选择你要连接的设备
* 当设备连接成功后，在工作区会出现以设备名为名称的目录节点（该节点映射了设备中的应用列表）


## 使用 IOT Editor 新建应用

* Shift + Alt + P 搜索运行 "IOT: Create new application"
* 输入应用实例名称和应用名称
* 设备中会新建出带有模板代码的新应用。同时VS Code工作区会出现该应用目录。
* 打开应用下面的入口文件(app.lua)就可以开始您的应用开发之旅

