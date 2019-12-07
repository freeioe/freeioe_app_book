
---

# MQTT 连接应用（上送数据）封装模块

> ***API_VER: 5***

本模块封装了MQTT上送数据的基础逻辑。帮助用户快速开发通过MQTT协议上送设备数据到云服务的应用。

应用示例:
* [阿里云接入](https://github.com/freeioe/freeioe_example_apps/blob/master/aliyun/app.lua)
* [百度云接入](https://github.com/freeioe/freeioe_example_apps/blob/master/baidu_cloud/app.lua)
* [广州联通云平台接入](https://github.com/freeioe/freeioe_example_apps/blob/master/telit/app.lua)

## initialize

构造 (同应用构造函数)

```lua
function initialize(name, sys, conf)
end
```

### MQTT 配置

配置信息 conf 可以通过包含以下信息，来配置MQTT连接认证相关的信息。

* client_id
  MQTT 客户端 ID
* username
  MQTT 连接认证用户名
* password
  MQTT 连接认证密码
* server
  MQTT 连接服务器地址（可以是域名和 IP)
* port
  MQTT 连接服务器端口
* clean_session
  MQTT 连接的 clean_session 标识
* enable_tls
  是否启用 TLS 安全认证
* tls_cert
  TLS 服务器证书文件路径（应用目录内的相对路径)
* tls_insecure
  是否启用非安全证书认证模式
* tls_ca_path
  TLS 服务器证书目录路径（应用目录内的相对路径)
* client_cert
  TLS 客户端证书文件路径（应用目录内的相对路径)
* client_key
  TLS 客户端密钥文件路径（应用目录内的相对路径)

### 其他配置

#### period

周期上送的周期时间(秒)，缺省是60秒。 小于1则不开启周期上送功能

#### ttl

变化传输的强制上传周期(秒)，数据不变，但是经过ttl的时间数据必须上传一次, 默认300秒

#### float_threshold

变化传输判断浮点数据变化的精度 (默认0.0000001)

#### data_upload_dpp

数据上传单包数据点数量上限。缺省是1024

#### data_upload_buffer

周期上送最多缓存数据点数量。缺省为10240

#### enable_data_cache

是否开启断线缓存。为1的时候开启断线缓存功能

#### cache_per_file

断线缓存单文件数据点数量。缺省为4096，范围是1024 ~ 4096

#### data_cache_limit

断线缓存文件数量上限。缺省为128，取值范围:1 ~ 256

#### data_cache_fire_gap

断线缓存上送数据间隔时间。单位毫秒, 缺省为1000，取值范围: 不小于1000

## 回调函数

模块派生应用需要提供以下回调函数处理数据

### 必选回调函数

#### on_publish_data

当未开启周期上送时，此接口会被调用，用以上传单点数据。

```lua
function app:on_publish_data(key, value, timestamp, quality)
end
```

#### on_publish_data_list

开启周期上送后，此接口用以上传多个数据（数据数组）

```lua
function app:on_publish_data_list(val_list)
end
```

#### pack_key

创建数据点唯一标识。

* 用以创建数据点唯一标识，默认本模块会以 ```<sn>/<input>/<prop>``` 进行编码，如有需要则重载此函数。
* 其次函数返回空(nil)表示该点数据需要忽略。可以用作忽略某种数据，或忽略某些设备的数据

```lua
function app:pack_key(app_src, device_sn, input, prop)
end
```

#### on_publish_data_em

当设备有紧急数据需要上送时调用此接口（同一紧急数据也会抄送至普通数据处理接口)

```lua
function app:on_publish_data_em(key, value, timestamp, quality)
end
```

#### on_publish_devices

当网关采集的设备信息有变动时此接口被调用（或MQTT刚连接成功时，此接口会被调用一次)。用以上送当前采集的设备信息（如设备描述等等）

```lua
function app:on_publish_devices(devices)
end
```

#### on_comm

通过此接口处理设备通讯报文

```lua
function app:on_comm(app_src, device_sn, dir, timestmap, ...)
end
```

#### on_event

通过此接口可以监听当前网关(FreeIOE)内部的设备事件

```lua
function app:on_event(app_src, device_sn, event_level, event_data, event_timestamp)
end
```

#### on_stat

通过此接口可以监听统计数据（详情参考统计接口的文档获取更详细信息）

```lua
function app:on_stat(app_src, device_sn, stat, prop, value, timestamp)
end
```

#### on_publish_cached_data_list

当开启MQTT模块的断线缓存功能时，此接口用以处理缓存数据。

```lua
function app:on_publish_cached_data_list(val_list)
end
```

#### mqtt_auth

此接口用以实现复杂的MQTT认证，需要返回以下信息

```lua
function app:mqtt_auth()
end
```

```lua
return {
	client_id = client_id,
	username = username,
	password = password,
	clean_session = true,
	host = host,
	port = port,
	enable_tls = enable_tls,
	tls_cert = "root_cert.crt",
}
```

#### mqtt_will

实现此接口会让模块在连接MQTT服务器成功时发送on will 消息，用以发布离线通知

```lua
function app:mqtt_will()
	return will_topic, will_data, will_qos, will_retained
end
```

#### on_mqtt_connect_ok

此接口会在连接成功时被调用，可用以发送上线通知以及订阅主题等操作

```lua
function app:on_mqtt_connect_ok()
end
```

#### on_mqtt_message

处理MQTT下行数据

```lua
function app:on_mqtt_message(msg_id, topic, payload, qos, retained)
end
```

#### on_mqtt_publish

本模块已经处理了qos为1的信息处理，此接口用以进行额外处理（如qos 2的信息处理等等)

```lua
function app:on_mqtt_publish(msg_id)
end
```

## connected

返回是否已经连接成功

```lua
function app:connected()
```

## connect

开启连接（仅用以调用disconnect()后再次发起连接，本模块启动后会自动开启连接)

```lua
function app:connect()
```

## disconnect

断开与MQTT服务器的连接

```lua
function app:disconnect()
```

## subscribe

订阅主题

```lua
function app:subscribe(topic, qos)
```

## unsubscribe

取消订阅主题

```lua
function app:unsubscribe(topic)
```

## publish

发布MQTT信息数据

```lua
function app:publish(topic, msg, qos, retained)
```

## compress

压缩数据（string类型), 返回压缩好的数据(string类型)。压缩算法为zip压缩

```lua
function app:compress(data)
```

## decompress

解压数据(string类型)

```lua
function app:decompress(data)
```
