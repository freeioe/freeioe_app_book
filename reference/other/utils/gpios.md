
----

# 控制设备的GPIO

用以帮助应用控制设备GPIO的输出。


### value
> function led:value(value)

设定GPIO的电平状态（value为nil时，获取当前电平状态信息)


### toggle
> funciton led:toggle()

切换输出状态，如果当前状态为高电平状态，则切换到低电平状态


### blink
> function led:blink(sec, dark_sec)

设定GPIO进入闪烁切换状态。 sec是GPIO高电平时长(浮点数), dark_sec是GPIO低电平时长。 闪烁会一直执行到取消闪烁


### cancel_blink
> function led:cancel_blink()

取消GPIO电平闪烁状态


### 使用示例：

``` lua

local gpios = require 'utils.gpios'

local pcie_reset = leds.pcie_reset --- 使用名称为pcie_reset的GPIO, /sys/class/gpio下的名称为pcie_reset的GPIO。

pcie_reset:value(1) -- 高电平
pcie_reset:value(0) -- 低电平

pcie_reset:blink(1, 1) -- 闪烁

```
