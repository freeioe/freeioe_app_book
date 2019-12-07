
----

# FreeIOE系统信息获取、设定模块

用以获取或设定FreeIOE系统信息

## id

获取系统连接云平台的ID，可与硬件序列号不一致

```lua
function ioe.id()
end
```

## hw_id

获取硬件序列号。

```lua
function ioe.hw_id()
end
```

### beta

获取当前设备的调试模式是否开启

```lua
function ioe.beta()
end
```

### auth_code

获取系统登录认证码，一般是从平台获取的认证码并保存在本地的字符串。

```lua
function ioe.auth_code()
end
```

### set_auth_code

设定系统登录认证码，从平台获取后，通过此接口设定

```lua
function ioe.set_auth_code()
end
```

### pkg_host_url

获取FreeIOE应用下载时使用的服务器地址

```lua
function ioe.pkg_host_url()
end
```

### set_pkg_host_url

设定应用下载使用的服务器地址。谨慎使用，如设定了不可用的地址，会导致系统升级、应用安装等等功能不可用。

```lua
function ioe.set_pkg_host_url(value)
end
```

### cnf_host_url

获取FreeIOE系统配置备份服务器地址。

```lua
function ioe.cnf_host_url()
end
```

### set_cnf_host_url

设定FreeIOE系统配置备份服务器地址。 谨慎使用，会应用系统的配置的备份功能。

```lua
function ioe.set_cnf_host_url(value)
end
```

### cfg_auto_upload

获取系统是否会自动备份配置(boolean)

```lua
function ioe.cfg_auto_upload()
end
```

### set_cfg_auto_upload

设定是否自动备份系统配置。

```lua
function ioe.set_cfg_auto_upload(value)
end
```

### time

获取系统时间，返回浮点数字，整数部分是秒数(Unix时间戳)，小数部分是毫秒值（只保留了两位精度)

```lua
function ioe.time()
end
```

### starttime

获取FreeIOE系统启动的时间。（非时长，非硬件启动时间，请使用sysinfo.uptime获取硬件启动时长)

```lua
function ioe.starttime()
end
```

### abort_prepare

FreeIOE 系统重启前的准备工作，如关闭所有应用

> *** API_VER: 5 ***

```lua
function ioe.abort_preapre()
end
```

### abort

FreeIOE系统重启（非操作系统重启)。 自动调用abort_prepare函数。 timeout为等待重启准备工作完成的时间长度，默认为5000, 单位是ms。

> *** API_VER: 5 ***

```lua
function ioe.abort(timeout)
end
```
