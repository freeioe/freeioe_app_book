
----

# 计时模块

```
API_VER: 6 
```

用以周期计时模块

### new
> function summation:new(callback, span, integral_time)

* callback <br> 回调函数，当时间到达计时周期时，会回调这个函数。
* span <br> 周期时间（秒）
* integral_time <br> boolean, 是否采取整点时间回调，否则会从当前时间开始计时(调用start的时间)。

回调函数的原型是
```lua
--- now 是当前时间
function(now)
end
```

当span是60， integral_time为true时，回调函数会在没分钟的0秒时回调callback函数。
模块只能尽可能满足这个整点时间要求，但是具体的时间有可能会晚于0秒时间。
具体时间差异要收到操作系统调度，以及使用此模块的应用进程内是否有其他比较耗时操作的影响。

### start
> funciton summation:start()

开启计时


### stop
> function summation:stop()

停止周期计时

