
---

# 二进制数据操作

Lua 构建和解析二进制数据有两种方式：

* 使用string.char, string.byte
* 使用string.pack, string.unpack


## 使用string.char, string.byte

***函数参考:***
* [string.byte](http://www.lua.org/manual/5.3/manual.html#pdf-string.byte)
* [string.char](http://www.lua.org/manual/5.3/manual.html#pdf-string.char)


***示例：***

```lua
local toInt16  = function(val)
    local val = (val + 65536) % 65536
    local hv = math.floor((val / 256) % 256) 
    local lv = math.floor(val % 256)
    return string.char(hv)..string.char(lv)
end
local fromInt16= function(data)
    local val = _M.uint16(data, index)
    val = ((val + 32768) % 65536) - 32768
    return val
end
```

## 使用string.pack, string.unpack

***函数参考:***
* [string.pack/unpack](http://www.lua.org/manual/5.3/manual.html#6.4.2)


***示例：***

```lua
local toInt16 = function(value)
    return string.pack(">i2", value)
end
local fromInt16 = function(data)
    return string.unpack(">i2", data)
end
```

# 位数据操作

Lua 5.3 原生支持位操作。[参考](http://www.lua.org/manual/5.3/manual.html#3.4.2)

