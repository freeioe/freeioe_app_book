---

# HTTP下载模块 (http.download)

本模块提供HTTP GET请求功能，支持自动重定向处理。

## 特性

- 简单的HTTP GET请求接口
- 自动处理301/302重定向
- 支持自定义请求头
- 支持查询参数
- 支持请求体内容

## 使用方法

```lua
local http_download = require 'http.download'

-- 基本GET请求
local status, headers, body = http_download.get('https://example.com', '/api/data')

-- 带查询参数
local status, headers, body = http_download.get('https://api.example.com', '/search', nil, {
    q = 'keyword',
    page = 1
})

-- 带自定义请求头
local status, headers, body = http_download.get('https://api.example.com', '/api/data', {
    Authorization = 'Bearer token123',
    ['User-Agent'] = 'MyApp/1.0'
})

-- 带请求体
local status, headers, body = http_download.get('https://api.example.com', '/api/data', nil, nil, 'request body')
```

## 接口说明

### get

执行HTTP GET请求

#### 函数原型

```lua
function http_download.get(host, url, header, query, content)
end
```

#### 参数说明

* host
  目标主机地址，格式如 `https://example.com` 或 `http://example.com:8080`
* url
  请求路径，如 `/api/data`
* header
  可选，请求头表
* query
  可选，查询参数表，会自动编码为URL参数
* content
  可选，请求体内容

#### 返回值

成功返回：
1. status - HTTP状态码
2. headers - 响应头表
3. body - 响应体内容

失败返回：
1. nil
2. error - 错误信息

#### 自动重定向

模块会自动处理301和302重定向：
- 如果Location头是相对路径（以`/`开头），会在同一主机上重定向
- 如果Location头是绝对路径，会解析新主机并发起新请求
- 重定向时会清除Host头，避免错误

#### 查询参数编码

查询参数会自动进行URL编码，支持所有字符类型，包括布尔值。
