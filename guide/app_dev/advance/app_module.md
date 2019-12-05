
---

# 应用本质

本章我们介绍 FreeIOE 应用在编程上的本质，有助于更好的理解和开发 FreeIOE 应用。

本质上 FreeIOE 应用是一个独立的 [Lua 模块](http://lua-users.org/wiki/ModulesTutorial)，且 FreeIOE 对此模块提出了以下的要求：

1. 需要具备 new 函数，来实例化模块对象
2. 实例化后的模块对象需要具备以下功能函数:
   1. start
   2. run
   3. close

## 模块实例化函数

FreeIOE 会使用下述方法来实例化应用模块：

```lua
local obj = m:new(app_name, sys_api, conf_obj)
```

> FreeIOE 框架加载应用的入口文件 app.lua， 来获取应用模块（m)。
> 之后 FreeIOE 框架会使用上述的 new 的方法来实例化应用对象

## 应用对象功能函数

### start 函数

此函数就是 FreeIOE 启动应用时会调用的函数，方便应用在启动时进行一些初始化工作。

注意：

我们不建议在此函数进行过于复杂（耗时）的逻辑。FreeIOE 如果无法尽快获取应用启动的结果，会导致用户在远程操作时因为等待超时而无法知道应用的具体情况。

```lua
function app:start()
end
```

### close 函数

当 FreeIOE 去停止应用时，会调用此函数来让应用完成逻辑停止和资源释放等操作。

如同 start 函数一样，停止函数也应该是尽量的快速完成其工作。过于长时间的等待，也可能导致用户失去等待的耐心。

```lua
function app:close(reason)
end
```

### run 函数

此函数是一个可选函数，是框架提供的一个默认的定时运行的函数入口。原型如下：

```lua
function app:run(tms)
    -- Your code here

    return 1000 --- FreeIOE will call run after 1000 ms.
end
```