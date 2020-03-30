
---

# 系统接口

FreeIOE框架提供的系统类型接口。

## log

记录应用日志。

```lua
function sys:log(level, ...)
end
```

level是字符串类型，可用值:trace, debug, info, notice, warning, error, fatal.

### 示例代码

```lua
sys:log("debug", "this is a log content")
```

## logger

```lua
function sys:logger()
end
```

获取logger实例。 实例包含:trace, debug, info, notice, warning, error, fatal.

### 示例代码

```lua
local log = sys:logger()
log:debug("this is a log content")
```

## dump_comm

记录应用报文。

```lua
function sys:dump_comm(sn, dir, ...)
```

sn是应用创建的设备序列号/为空时代表设备无关报文。参考[device](device.md) 中的[dump_comm](device.md#dump_comm)函数。 dir是报文方向： IN, OUT。

## fire_event

记录应用事件。

```lua
function sys:fire_event(sn, level, type, info, data, timestamp)
end
```

### 参数说明

* sn
  应用创建的设备序列号。
* level
  事件等级的整数
* type
  事件类型(如果是字符串类型则是自定义类型)
* info
  事件描述字符串
* data
  时间附带数据
* timestamp
  时间戳。

## fork

创建独立携程,入口执行函数 func.

```lua
function sys:fork(func, ...)
end
```

### 示例代码

```lua
sys:fork(function(a)
    print(a)
end, 1)
```

> 其中a是传入的参数1

## timeout

创建延迟执行携程

```lua
function sys:timeout(ms, func)
end
```

ms 为延迟时间（单位是毫秒）, func 为携程入口函数。

## cancelable_timeout

创建可以取消的延迟携程，

```lua
function sys:cancelable_timeout(ms, func)
end
```

返回对象是取消函数

#### 示例代码

```lua
local timer_cancel = sys:cancelable_timeout(...)
timer_cancel()
```

## exit

应用退出接口。请谨慎使用。

```lua
function sys:exit()
end
```

## abort

系统退出接口，调用此接口会导致 FreeIOE 系统退出。 请谨慎调用。

```lua
function sys:abort()
end
```

## now

返回操作系统启动后的时间计数。 单位是微妙，最小有效精度是10毫秒。

```lua
function sys:now()
end
```

## time

获取系统时间，单位是秒，并包含两位小数的毫秒。

```lua
function sys:time()
end
```

## start_time

系统启动的UTC时间，单位是秒。

```lua
function sys:start_time()
end
```

## yield

交出当前应用对CPU的控制权。相当与 ```sys:sleep(0)```。

```lua
function sys:yield()
end
```

## sleep

挂起当前应用， ms是挂起时常，单位是毫秒

```lua
function sys:sleep(ms)
end
```

## data_api

获取数据接口，参考[api](api.md)

```lua
function sys:data_api()
end
```

## self_co

获取当前运行的携程对象

```lua
function sys:self_co()
```

## wait

挂起当前携程，结合 ```sys:wakeup``` 使用

```lua
function sys:wait()
end
```

## wakeup

唤醒一个被 ```sys:sleep``` 或 ```sys:wait``` 挂起的携程。

```lua
function sys:wakeup(co)
end
```

## app_dir

获取当前应用所在的目录。

```lua
function sys:app_dir()
end
```

## app_sn

获取当前应用的序列号。

```lua
function sys:app_sn()
end
```

## get_conf

获取应用配置，default_config 默认配置

```lua
function sys:get_conf(default_config)
end
```

## set_conf

设定应用配置

```lua
function sys:set_conf(config)
end
```

## conf_api

获取云配置服务接口。 具体参考[conf_api](conf_api.md)

```lua
function sys:conf_api(conf_name, ext, dir)
end
```

## version

获取应用版本号。返回应用 ID 和应用版本

```lua
function sys:version()
end
```

### 示例代码

```lua
local app_id, version = sys:version()
print(app_id, version)
```

## gen_sn

生成独立的设备序列号，dev_name 为设备名称，必须指定。

```lua
function sys:gen_sn(dev_name)
end
```

## hw_id

获取FreeIOE设备序列号

```lua
function sys:hw_id()
end
```

## id

获取 FreeIOE 连接云平台所用的序列号(此ID可不同与设备序列号)

```lua
function sys:id()
```

## req

发送同步请求，相应函数为```app.response```或者```app.on_req_<msg>```函数

```lua
function sys:req(msg, ...)
end
```

## post

发送异步请求，相应函数为```app.accept```或者```app.on_post_<msg>```函数

```lua
function sys:post(msg, ...)
end
```

## cloud_post

请求云平台连接配置接口

*** API_VER: 5 ***

```lua
function sys:cloud_post(func, ...)
end
```

### 配置项(func)

* enable_data_one_short
* enable_event
* download_cfg
* upload_cfg
* fire_data_snapshot
* batch_script

## cfg_call

请求FreeIOE系统配置服务接口：

*** API_VER: 5 ***

```lua
function sys:cfg_call(func, ...)
end
```

### 配置项(func)

* SAVE

## set_event_threshold

设定应用发送事件的数量限制（每分钟)，默认情况下应用每分钟最多只能发送20条事件。取值范围是0~127,超过限制的事件会被 FreeIOE 框架丢弃。

```lua
function sys:set_event_threshold(count_per_min)
```

## cleanup

应用清理接口(会自动被调用，请勿使用)

```lua
function sys:cleanup()
end
```
