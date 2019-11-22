
---

# 数据流缓存模块

> *** API_VER: 5 ***

本模块封装了数据流缓存模块。帮助用户快速开发数据解析类应用。


## 模块函数

### initialize
> function buffer:initialize(max_len)

初始化，指定最大缓存长度。

### concat
> function buffer:concat()

获取缓存所有数据（拼接后)

### find
> function buffer:find(sk, ek)

从缓存中查找匹配的数据，sk是数据头字符串，ek是结束字符串。成功返回数据和数据长度，失败返回nil,err

### pop
> function buffer:pop(len)

移除数据长度，查找到匹配数据后，从当前缓存中删除数据时使用。

### len
> function buffer:len()

获取缓存数据总长度

### droped
> function buffer:droped()

获取被丢弃的无效数据长度


### append
> function buffer:append(data)

追加数据流

### clean()

清除当前数据数据（丢弃)

