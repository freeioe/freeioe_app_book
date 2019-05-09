
---

# MQTT 连接应用（上送数据）封装模块


本模块封装了MQTT上送数据的基础逻辑。帮助用户快速开发通过MQTT协议上送设备数据到云服务的应用。


## 构造 (同应用构造函数)
> function initialize(name, sys, conf)

### 配置信息可以通过包含以下信息，来进行MQTT连接认证：
> client_id: MQTT Client ID
> username: MQTT User id
> password: MQTT User password
> server: MQTT Broker address (domain or ip)
> port: MQTT Broker port
> enable_tls: MQTT TLS enable or not
> tls_cert: MQTT Server cert file (if TLS enabled)

### 其他配置：

#### 周期上送的周期时间(秒)
> period

缺省是60秒。 小于1则不开启周期上送功能

#### 变化传输的强制上传周期(秒)
> ttl

变化传输的强制上传周期（数据不变，但是经过ttl的时间数据必须上传一次, 默认300秒)

#### 变化传输的浮点精度
> float_threshold

变化传输判断浮点数据变化的精度 (默认0.0000001)

#### 数据上传单包数据点数量上限
> data_upload_dpp

缺省是1024

#### 周期上送最多缓存数据点数量
> data_upload_buffer

缺省为10240

#### 是否开启短线缓存
> enable_data_cache

为1的时候开启断线缓存功能

#### 断线缓存单文件数据点数量
> cache_per_file

缺省为4096，范围是1024 ~ 4096

#### 断线缓存文件数量上限
> data_cache_limit

缺省为128，取值范围:1 ~ 256

#### 断线缓存上送数据间隔时间
> data_cache_fire_gap

单位毫秒, 缺省为1000，取值范围: 不小于1000

## 模块派生应用需要提供以下回调函数处理数据

### 必选回调函数
> function app:on_publish_data
> function app:on_publish_data_list

### 可选回调函数
> function app:pack_key
> function app:on_publish_data_em
> function app:on_publish_devices
> function app:on_comm
> function app:on_event
> function app:on_stat
> function app:on_publish_cached_data_list
> function app:mqtt_auth
> function app:mqtt_will
> function app:on_mqtt_connect_ok
> function app:on_mqtt_message
> function app:on_mqtt_publish


### 回调函数说明

#### on_publish_data
> function app:on_publish_data(key, value, timestamp, quality)
>

当未开启周期上送时，此接口会被调用，用以上传单点数据。

#### on_publish_data_list
> function app:on_publish_data_list(val_list)
>	

开启周期上送后，此接口用以上传多个数据（数据数组）

#### pack_key
> function app:pack_key(app_src, device_sn, input, prop)
>

用以创建数据点唯一标识，默认本模块会以 <sn>/<input>/<prop>进行编码，如有需要则重载此函数。次函数返回空(nil)表示该点数据需要忽略。

#### on_publish_data_em
> function app:on_publish_data_em(key, value, timestamp, quality)
>

当设备有紧急数据需要上送时调用此接口（同一紧急数据也会抄送至普通数据处理接口)

#### on_publish_devices
> function app:on_publish_devices(devices)
>

当网关采集的设备信息有变动时此接口被调用（或MQTT刚连接成功时，此接口会被调用一次)。用以上送当前采集的设备信息（如设备描述等等）


#### on_comm
> function app:on_comm(app_src, device_sn, dir, timestmap, ...)
>
通过此接口处理设备通讯报文

#### on_event
> function app:on_event(app_src, device_sn, event_level, event_data, event_timestamp)
>

通过此接口可以监听当前网关(FreeIOE)内部的设备事件

#### on_stat
> function app:on_stat(app_src, device_sn, stat, prop, value, timestamp)
>

通过此接口可以监听通讯统计数据（详情参考统计接口的文档获取更详细信息）

#### on_publish_cached_data_list
> function app:on_publish_cached_data_list(val_list)
>

当开启MQTT模块的断线缓存功能时，此接口用以处理缓存数据。

#### mqtt_auth
> function app:mqtt_auth()
> 

此接口用以实现复杂的MQTT认证，需要返回以下信息
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
> function app:mqtt_will()
>	return will_topic, will_data, will_qos, will_retained

实现此接口会让模块在连接MQTT服务器成功时发送on will 消息，用以发布离线通知

#### on_mqtt_connect_ok
> function app:on_mqtt_connect_ok()
>

此接口会在连接成功时被调用，可用以发送上线通知以及订阅主题等操作

#### on_mqtt_message
> function app:on_mqtt_message(msg_id, topic, payload, qos, retained)
>

处理MQTT下行数据

#### on_mqtt_publish
> function app:on_mqtt_publish(msg_id)
>

本模块已经处理了qos为1的信息处理，此接口用以进行额外处理（如qos 2的信息处理等等)

## 模块提供以下功能函数

### connected
> function app:connected()
>

返回是否已经连接成功

### connect
> function app:connect()
>

开启连接（仅用以调用disconnect()后再次发起连接，本模块启动后会自动开启连接)

### disconnect
> function app:disconnect()
> 

断开与MQTT服务器的连接

### subscribe
> function app:subscribe(topic, qos)
>

订阅主题

### unsubscribe
> function app:unsubscribe(topic)
>

取消订阅主题

### publish
> function app:publish(topic, msg, qos, retained)
> 

发布MQTT信息数据

### compress
> function app:compress(data)
>

压缩数据（string类型), 返回压缩好的数据(string类型)。压缩算法为zip压缩

### decompress
> function app:decompress(data)
>

解压数据(string类型)

