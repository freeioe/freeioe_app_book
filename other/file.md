---

# 文件操作


## Lua 读写文件:

* Lua 原生的io操作
* 读取CSV文件
* 读取INI文件


### 普通文件读写

参考：[io.open](http://www.lua.org/manual/5.3/manual.html#pdf-io.open)


### JSON 文件读写

FreeIOE 内置以下模块:

* [lua-cjson](https://github.com/cloudwu/lua-cjson)
* [json.lua](https://github.com/rxi/json.lua)


### CSV文件读写

FreeIOE 内置以下模块:

* [ftcsv](https://github.com/FourierTransformer/ftcsv)
* [lcsv](https://github.com/daelvn/lcsv)


### INI文件读写

FreeIOE 内置以下模块:

* [inifile](http://docs.bartbes.com/inifile)
* [LIP](https://github.com/Dynodzzo/Lua_INI_Parser)


### XML文件读写

推荐使用以下模块(FreeIOE 未集成任何XML模块)：

1. [xml2lua](https://github.com/manoelcampos/xml2lua) 只解析XML
2. [SLAXML](https://github.com/Phrogz/SLAXML) 解析生成XML
3. [luaxml]( https://github.com/natnat-mc/luaxml) 解析生成XML

更多XML相关模块: [lua-users](http://lua-users.org/wiki/LuaXml)


## LuaFileSystem

FreeIOE内置了LuaFileSystem用以进行文件系统操作。[文档](http://keplerproject.github.io/luafilesystem/)
