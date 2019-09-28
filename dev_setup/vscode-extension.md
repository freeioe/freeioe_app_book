
---

# 使用 VS Code 开发 FreeIOE 应用

VS Code 是微软开发提供的一款非常灵活的编辑器软件。 FreeIOE 提供了 VS Code的插件，让开发者可以方便快速的进行应用开发和调试。

## VS Code 下载

可以从微软网站下载VS Code。[地址](https://code.visualstudio.com/)


## IOT Editor 插件安装

运行VS Code，打开插件列表选项页，搜索并安装"IOT Editor"插件。

> 另：建议安装Lua语言插件: LuaCoderAssist


### 插件功能列表

* 可同时连接多个FreeIOE网关设备。
* 自动映射设备的应用源码，可同步修改设备中的应用源码。
* 在输出窗口输出设备日志，设备报文
* 查看设备的事件列表
* 查看应用的配置信息


### IOT Editor 界面功能区

![IOT Editor 功能区](assets/vscode_editor.png "插件功能区")


## IOT Editor 插件使用

### 连接设备

* 在本地磁盘中创建一个新的文件夹
* 使用VS Code打开新建的文件夹
* 从文件(F)菜单中选择"将工作去另存为"，保存当前工作区文件
* Shift + Alt + P 搜索运行 "IOT: Setup editor workspace"
* 在自动打开的freeioe_devices.json的devices节点下选择一个节点（如Device1节点)
* 将其中的"host"修改为真实的设备IP
* Shift + Alt + P 搜索运行 "IOT: Connect to freeioe device"
* 选择刚才修改的节点名称 （如Device1)
* 当设备连接成功后，在资源管理器的工作区下，会出现以设备名为名称的目录节点（该节点映射了设备中的应用列表）

> 一般情况下，请勿修改freeioe_devices.json文件中的devices节点下的端口，默认情况下设备的编程端口为8818


### 新建应用

使用IOT Editor可以创建一个全新的应用，步骤如下

* Shift + Alt + P 搜索运行 "IOT: Create new application"
* 输入应用实例名称和应用名称
* 设备中会新建出带有模板代码的新应用。同时VS Code工作区会出现该应用目录。
* 打开应用下面的入口文件(app.lua)就可以开始您的应用开发之旅


### 下载应用包

当应用代码开发完毕，通过"Download Application"右键菜单下载应用包:

1. 文件浏览器窗口的应用目录点击右键
2. 设备列表窗口的应用节点点击右键

IOT Editor会从设备中下载打包好的应用包到指定的目录。 下载完应用包，即可在应用中心上传和发布此应用。

> 请务必手工修改version文件，确保版本和应用中心上传时的版本一致。
