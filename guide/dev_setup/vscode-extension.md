
---

# 使用 VS Code 开发应用

VS Code 是微软开发提供的一款非常灵活的编辑器软件。 我们提供了 VS Code的插件，让开发者可以方便快速的进行 FreeIOE 应用开发和调试。

![IOT Editor](assets/vscode_arch.png)

## VS Code 下载

可以从微软网站下载VS Code。[地址](https://code.visualstudio.com/)

## 应用开发插件

运行VS Code，打开插件列表选项页，搜索并安装"IOT Editor"插件。

> 另：建议安装Lua语言插件: LuaCoderAssist

### 插件功能列表

* 连接 FreeIOE 网关设备
* 自动映射设备的应用源码，同步修改设备中的应用源码
* 启动、停止 FreeIOE 应用
* 显示 FreeIOE 应用的配置信息
* 显示输出来自网关设备的日志、报文
* 显示网关设备的事件列表


### IOT Editor 界面功能区

![IOT Editor 功能区](assets/vscode_editor.png "插件功能区")

#### 资源管理器

插件会将连接上的设备中的应用映射到资源管理器中，是以设备名称为节点的目录中。

#### FreeIOE Devices 窗口

此窗口显示本地配置的要连接的设备，并在连接成功后显示设备中的应用实例列表，以及应用当前的运行状态

#### Events 窗口

此窗口显示设备内的事件列表（缓存、非全部）

#### 文件编辑区

在资源管理器选择要编辑的应用的代码文件，该文件会被 VS Code 打开并出现在文件编辑区

#### 输出窗口

* IOT EDITOR
  显示插件的一些输出内容，为方便查找 IOT Editor 插件问题
* IOT 设备报文
  显示连接中的设备的通讯报文
* IOT 设备日志
  显示连接中的设备的日志信息

## IOT Editor 插件使用

### 连接设备

操作步骤如下：

1. 在本地磁盘中创建一个新的文件夹
2. 使用 VS Code 打开新建的文件夹
3. 从文件(F)菜单中选择"将工作区另存为"，保存当前工作区文件
4. Shift + Alt + P 搜索运行 "IOT: Setup editor workspace"
5. 在自动打开的 freeioe_devices.json 的 devices 节点下选择一个节点（如Device1节点)
6. 将其中的 "host" 修改为真实的设备IP
7. Shift + Alt + P 搜索运行 "IOT: Connect to freeioe device"
8. 选择刚才修改的节点名称 （如 Device1 )
9. 当设备连接成功后，在资源管理器的工作区下，会出现以设备名为名称的目录节点（该节点映射了设备中的应用列表）

注意：

> 一般情况下，请勿修改设备端口(port)，默认情况下设备的编程端口为8818
> 如FreeIOE的管理密码被修改，请对应修改password为正确的密码

### 新建应用

使用 IOT Editor 可以创建一个全新的应用，步骤如下

1. Shift + Alt + P 搜索运行 "IOT: Create new application"
2. 输入应用实例名称和应用名称
3. 设备中会新建出带有模板代码的新应用。同时 VS Code 工作区会出现该应用目录。
4. 打开应用下面的入口文件(app.lua)就可以开始您的应用开发之旅


### 下载应用包

当应用代码开发完毕，通过 "Download Application" 右键菜单下载应用包:

1. 文件浏览器窗口的应用目录点击右键
2. 设备列表窗口的应用节点点击右键

IOT Editor会从设备中下载打包好的应用包到指定的目录。 下载完应用包，即可在应用中心上传和发布此应用。

> 请务必手工修改version文件，确保版本和应用中心上传时的版本一致。
