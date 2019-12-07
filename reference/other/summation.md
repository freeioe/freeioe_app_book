
----

# 累计量计算模块

```
API_VER: 6
```

用以计算周期内累计量，使用文件进行存储。且处理当数值重置时进行累加，而不是使用重置后的数值。

例如，通过获取网卡的当前收发字节数，来统计一天的字节数，当网卡被重启、系统重启后能继续累加字节数。且当零点时重置计数。

## new

初始化模块对象

```lua
function summation:new(opt)
end
```

opt包含以下参数：
| --- | --- | --- |
| file | boolean | 是否使用文件存储(不使用文件存储，则系统重启计数会丢失) |
| span | string | 计数的周期，可用周期: year, month, day, hour, minute |
| save_span | number | 文件保存周期(秒)，建议使用 60 * 5。太短的文件保存周期会导致过度使用设备存储，伤害存储寿命。 |
| key | string | 系统内独立的统计名称，建议使用应用实例名 + 基数简称 |
| path | string | 可选，文件保存路径，建议使用sysinfo.data_dir()获取地址 |
| on_reset | function | 可选，当计数周期完毕，重置累计量前，会回调次函数 |

## add

累加计数，会在当前的累计数值上加上参数value的数值。
key 是累计属性，即本对象支持多个累计属性。（当然所有累计属性使用同一个计数周期）

```lua
function summation:add(key, value)
end
```

## set

设定当前获取的累计值，如果当前累计值小于之前传入的累计值，则会认为累计值已被重置，会在之前的累计值的基础上加上当前的累计值。

例如。上一次调用的值为10000,而本次调用为100时，调用get出来的累计值会是 10100

```lua
function summation:set(key, value)
end
```

## get

获取统计出来的累计值

```lua
function summation:get(key)
end
```

## reset

重置所有累计数值。

```lua
function summation:reset()
end
```

## set_reset_cb

设定on_reset回调函数，同new函数的opt中的on_reset函数 (如果opt中有on_reset函数，会使用callback替换opt中的on_reset)

```lua
function summation:set_reset_cb(callback)
end
```

## save

保存数据，当您的应用被停止时，建议调用一次本函数进行数据保存。避免保存周期还未执行导致的新增的计数丢失。

```lua
function summation:save()
end
```
