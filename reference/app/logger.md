
---

# 日志接口

系统日志接口的封装模块。 提供一下函数：

## log

日志接口（内部接口）

### 函数原型

```lua
function logger:log(level, ...)
end
```

## trace

输出trace等级日志

### 函数原型

```lua
function logger:trace(...)
end
```

## debug

输出调试等级日志

### 函数原型

```lua
function logger:debug(...)
end
```

## info

输出信息等级日志

### 函数原型

```lua
function logger:info(...)
end
```

## notice

输出通知等级日志

### 函数原型

```lua
function logger:notice(...)
end
```

## warning

输出警告等级日志

### 函数原型

```lua
function logger:warning(...)
end
```

## error

输出错误等级日志

### 函数原型

```lua
function logger:error(...)
end
```
