
---

# 数据流缓存模块

> *** API_VER: 5 ***

本模块封装了数据流缓存模块。帮助用户快速开发数据解析类应用。

## initialize

实例构造函数

#### 函数原型

```lua
function buffer:initialize(max_len)
```

#### 参数说明

* max_len
  指定最大缓存长度。超过此长度后，最旧的数据会被丢弃。

## concat

获取缓存所有数据（拼接后)，结果为字符串。

#### 函数原型

```lua
function buffer:concat()
```

## find

从缓存中查找匹配的数据

#### 参数说明

```lua
function buffer:find(sk, ek)
```

* sk
  sk是数据头字符串
* ek
  ek是结束字符串

成功返回数据和数据长度，失败返回nil,err

## pop

移除数据长度，查找到匹配数据后，从当前缓存中删除数据时使用。

#### 函数原型

```lua
function buffer:pop(len)
```

#### 参数说明

* len
  移除的数据的长度。

## len

获取缓存数据总长度

#### 函数原型

```lua
function buffer:len()
```

## droped

获取被丢弃的无效数据长度

#### 函数原型

```lua
function buffer:droped()
```

## append

追加数据流

#### 函数原型

```lua
function buffer:append(data)
```

#### 参数说明

* data
  追加的数据流数据

## clean

清除当前数据数据（丢弃)

#### 函数原型

```lua
function buffer:clean()
```
