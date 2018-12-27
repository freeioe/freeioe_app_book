
---

# Lua模块

## FreeIOE内置的模块(第三方)：

| 模块 | 说明|
| :--- | :--- |
| [SocketChannel](https://github.com/cloudwu/skynet/wiki/SocketChannel) | TCP 套接字通讯模块 |
| [ittner/lua-iconv](https://github.com/ittner/lua-iconv) | iconv 字符转码模块 |
| [cloudwu/lsocket](https://github.com/cloudwu/lsocket) | Socket封装模块，支持Unix本地Socket。 |
| [kooiot/lua-mosquitto](https://github.com/kooiot/lua-mosquitto) | libmosquitto的封装模块(MQTT) |
| [brimworkds/lua-zlib](https://github.com/brimworks/lua-zlib) | zlib封装模块 |
| [Lua-cURL/Lua-cURLv3](https://github.com/Lua-cURL/Lua-cURLv3) | libCURL 模块 |
| [keplerproject/lfs](http://keplerproject.github.io/luafilesystem/) | Lua 文件系统接口模块 |
| [keplerproject/md5](https://github.com/keplerproject/md5) | MD5 计算模块 |
| [user-none/lua-hashings](https://github.com/user-none/lua-hashings) | 哈希模块(sha1,sha256,sha512,md5,crc32等等) |
| [user-none/nums](https://github.com/user-none/lua-nums) | Lua超大整数，无符号整数支持 |
| [srdgame/librs232](https://github.com/srdgame/librs232) | 串口接口使用模块 |
| [lpeg](http://www.inf.puc-rio.br/~roberto/lpeg/) | Parsing Expression Grammars For Lua |
| [srdgame/bcd.lua](http://github.com/srdgame/bcd.lua) | BCD解析(支持指定格式) |
| [aiq/basexx](https://github.com/aiq/basexx) | 二进制数据转码 |
| [Tieske/date](https://github.com/Tieske/date) | Lua 日期&时间模块 |
| [ftcsv](https://github.com/FourierTransformer/ftcsv) | CSV文件解析 |
| [lcsv](https://github.com/daelvn/lcsv) | CSV文件解析 |
| [srdgame/lua-cjson](https://github.com/srdgame/lua-cjson) | JSON解析(cjson) |
| [json.lua](https://github.com/rxi/json.lua) | 纯lua实现的json解析模块 |
| [LIP](https://github.com/Dynodzzo/Lua_INI_Parser) | INI文件解析 |
| [inifile](http://docs.bartbes.com/inifile) | INI文件解析 |
| [kikito/middleclass](https://github.com/kikito/middleclass) | Lua 面向对象(OO) 帮助模块 |
| [kikito/stateful.lua](https://github.com/kikito/stateful.lua) | Stateful classes for Lua |
| [kyleconroy/lua-state-machine](https://github.com/kyleconroy/lua-state-machine) | A finite state machine lua micro framework |
| [Skycrab/skynet_websocket](https://github.com/Skycrab/skynet_websocket) |  skynet websocket(lua) |
| [Tieske/uuid](https://github.com/Tieske/uuid) | 纯Lua实现的UUID模块 |
| [moteus/lua-log](https://github.com/moteus/lua-log) | 异步日志模块 |

## FreeIOE 提供的模块：

| 模块 | 说明 |
| :--- | :--- |
| SerialChannel | 接口模式同SocketChannel，区别是串口通道只支持SocketChannel中的模式1(即一问一答模式) |
| cyclebuffer | 循环缓存模块(设定最大缓存条目后，会自动丢弃最老数据) |
| periodbuffer | 批次数据整理模块 |
| summation | 累计计数模块(适用于网络使用量计算，涉及重启基数归零后的重置计算等等) |
| cov | 变化处理模块，可以用数据变化传输 |
| ubus/ubox | ubus消息解析模块 |
| restful | RestFul API模块(使用skynet http模块实现) |


## utils(模块/目录)

| 模块 | 说明 |
| :--- | :--- |
| gcom | 调用gcom脚本获取信号强度，SIM卡信息等 |
| led | 控制设备led灯 |
| log | 日志模块(使用lua-log模块实现) |
| process_monitor | 调用process monitor监控运行其他进程 |
| services | 使用系统(Linux) 服务来监控运行其他进程 |
| retry | 限制次数的自动重试 |
| sysinfo | 系统信息获取帮助模块 |


## 非集成扩展模块

| 名称 | 地址 | 说明 |
| :--- | :--- | :--- |
| opcua | [symgrid/open62541-lua](https://github.com/symgrid/open62541-lua) | OPC UA(open62541)协议库的Lua扩展模块 |
| snap7 | [srdgame/lua-snap7](https://github.com/srdgame/lua-snap7) | Snap7协议库的Lua扩展模块(Siemens PLC) |
| plctag | [srdgame/libplctag](https://github.com/srdgame/libplctag) | Allen-Bradley PLC 协议库的Lua扩展模块 |


> 可以从[这里](https://github.com/freeioe/freeioe_prebuild_exts) 获取二进制文件  
> 如何在FreeIOE应用中使用非集成模块，[参考](https://github.com/freeioe/freeioe_example_apps/blob/master/opcua_client/depends.txt)  
> 
