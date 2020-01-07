
---

# 应用配置可视化

FreeIOE 在同一个应用的不同实例上，使用不同的配置信息（JSON数据），如使用不同的串口、协议类型、产品型号等等。配置信息会在应用对象实例化时，作为参数传给应用对象的构造函数。

``` lua
function app:initialize(name, sys, conf)
    --- name 实例名
    --- sys 系统接口
    --- conf 就是应用指定的配置信息
end
```

冬笋云提供的应用安装时，提供用户友好的、可视化的应用实例配置信息页面。

应用可视化配置是开发者可以通过应用设置页面进行自定义。设定页面如下:

![AppTree](images/cloud_app_settings.png)

该页面包含两个信息：

* 可视化配置描述 (JSON)
  描述可视化面板包含的配置项，以及配置项的可视化形式
* 应用初试值 (JSON)
  提供一个默认的配置信息

## 应用可视化配置的 JSON 格式说明

### 示例

下面是一个可视化的配置 JSON 示例：

``` json
[{
        "name": "protocol",
        "desc": "通讯协议",
        "type": "dropdown",
        "values": [
            "tcp",
            "rtu"
        ]
    },
    {
        "name": "Link_type",
        "desc": "链路类型",
        "type": "dropdown",
        "depends": {
            "socket": "socket",
            "serial": "serial"
        },
        "values": [
            "socket",
            "serial"
        ]
    },
    {
        "name": "socket",
        "desc": "TCP端口设定",
        "type": "tcp_client"
    },
    {
        "name": "serial",
        "desc": "串口设定",
        "type": "serial"
    },
    {
        "name": "tpls",
        "desc": "设备模板选择",
        "type": "templates"
    },
    {
        "name": "devs",
        "desc": "设备列表",
        "type": "table",
        "cols": [{
                "name": "addr",
                "desc": "Modbus地址",
                "type": "number"
            },
            {
                "name": "tpl",
                "desc": "设备模板",
                "type": "template"
            },
            {
                "name": "name",
                "desc": "设备名称",
                "type": "string"
            },
            {
                "name": "desc",
                "desc": "设备描述",
                "type": "string"
            },
            {
                "name": "sn",
                "desc": "设备序列号",
                "type": "string"
            }
        ]
    },
    {
        "name": "encryption",
        "desc": "证书选择",
        "type": "section",
        "child": [{
                "name": "cert",
                "desc": "UA证书(可选)",
                "type": "text",
                "value": "certs/cert.der"
            },
            {
                "name": "key",
                "desc": "KEY文件(可选)",
                "type": "string",
                "value": "certs/key.der"
            }
        ]
    }
]
```

### JSON 格式说明

1. 可视化 JSON 模板信息的根节点必须是一个数组。
2. 由应用可视化生成的数据结果也是 JSON,而其根节点是字典。
3. 可视化组件目前支持的类型(type)有：
    * boolean - 选择框
    * number - 数字输入项
    * string - 字符串输入框
    * text - 文本输入框(多行)
    * dropdown - 下拉框
    * tcp_client - TCP 客户端选项设定 （包含服务器 IP，服务器端口两部分）
    * serial - 串口配置选项 （包含波特率、数据位、停止位等等)
    * section - 数据组 (用来将一组基础类型进行分组)
    * fake_section - 数据组 (用来将一组基础类型进行分组， 但不会生成字节点)
    * table - 数据表格 （必须包含有 cols 字节点）
    * templates - 应用模板选择控件
    * template - 模板选择下拉框 (只能用在 table 组内)
4. 可视化节点的 name 是结果 JSON 中的 KEY （fake_section 除外)
5. 可视化节点的 default 属性为控件的默认值

#### boolean/number/string/text

此四种为基础控件，可设定的属性有：

1. name: 在结果中的 key 名称
2. type: 类型
3. default: 默认值

#### section

数据组（数据区域）的作用将控件进行区域分割，section 下的 child 配置项的内容会出现在 section 的字节点下。

#### fake_section

数据组（数据区域）的作用仅仅是将控件进行显示区域分割，并不会生成子节点。其 child 下的配置数据会出现在 JSON 根数据下。

#### dropdown

下拉框 UI 控件,较之基础控件多出以下属性：

1. values: 字符串数组(对象数组)描述下拉框可选择项\
    当为对象数组时，对象需要包含 name value 两个字段，其中 name 字段用以显示，value 字段用以生成 JSON 配置（depends 存在的话也是基于 value 字段进行判断)
2. depends: 字典描述 values 下的可选择项所影响的 UI 控件名称(顶级控件或者本区域内的控件）

#### tcp_client

TCP客户端连接属性配置，包括:

1. host: 服务器地址
2. port: 服务器端口
3. nodelay: 是否开启 NODELAY 模式

#### serial

串口端口属性设定，包括：

1. port: 串口名称（如 /dev/ttyS1)
2. baudrate: 波特率
3. data_bit: 数据位
4. stop_bits: 停止位
5. parity: 校验
6. flow_control: 流控

#### templates

应用模板选择列表，包括：

1. key: 数组唯一 ID
2. id: 应用模板在平台的 ID
3. name: 本地名称
4. description: 本地描述
5. version: 应用模板的版本

#### template

模板选择下拉框，目前只能出现在 table 空间内，可选择内容为 templates 节点的数据

#### table

表格控件，自定义列的类型可以是：number/string/template/dropdown。 同时列的属性节点名称(name)应避免使用“key"，此名称保留给 UI 空间作为数组 ID 来使用

注: 表格中的 dropdown 控件不再支持 depends 功能。
