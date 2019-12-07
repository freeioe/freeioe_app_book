
----

# 工具函数集

一些有用的函数集

## escape

使用html转义规则对str进行转义。如 &转义成为 &amp;amp;

```lua
> function helper.escape(str)
end
```

## encode_query_string

转码URL的查询参数。t是参数的table，如 {username="xxx",password="XXXX"}。sep是分割符，默认是&
输出：username=xxx&password=XXXX

```lua
> funciton helper.encode_query_string(t, sep)
end
```

## md5sum_lua

计算文件的MD5值（纯Lua实现，速度稍慢，请勿用作计算较大文件的MD5)

```lua
> function helper.md5sum_lua(file_path)
end
```

## md5sum

使用系统md5sum计算文件的MD5值，可以用做较大文件的MD5计算。如果当前操作系统中没有md5sum指令，则会使用md5sum_lua函数进行计算。

```lua
> function helper.md5sum(file_path)
end
```
