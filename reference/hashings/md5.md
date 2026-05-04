---

# MD5哈希算法 (hashings.md5)

MD5（Message-Digest Algorithm 5）是一种广泛使用的哈希函数，产生128位（16字节）的哈希值。

⚠️ **安全警告**：MD5已被证明存在碰撞攻击，**不应再用于安全目的**（如密码存储、数字签名等）。仅适用于非安全场景的数据完整性校验。

## 特性

- **摘要长度**：128位（16字节）
- **块大小**：512位（64字节）
- **输出格式**：通常表示为32个十六进制字符

## 使用方法

```lua
local md5 = require 'hashings.md5'

-- 创建MD5哈希对象并计算哈希
local h = md5:new('Hello World')
print(h:hexdigest())  -- 输出: B10A8DB164E0754105B7A99BE72E3FE5

-- 流式处理
local h = md5:new()
h:update('First part ')
h:update('Second part')
print(h:hexdigest())

-- 获取二进制输出
local h = md5:new('Hello World')
local binary = h:digest()  -- 16字节二进制数据
```

## 接口说明

### new

创建MD5哈希对象

#### 函数原型

```lua
function md5:new(data)
end
```

#### 参数说明

* data
  可选，初始数据字符串

#### 返回值

返回MD5哈希对象

### update

更新哈希计算状态

#### 函数原型

```lua
function md5:update(data)
end
```

#### 参数说明

* data
  要添加到哈希计算中的数据字符串

### digest

获取二进制哈希摘要

#### 函数原型

```lua
function md5:digest()
end
```

#### 返回值

返回16字节的二进制字符串

### hexdigest

获取十六进制哈希摘要

#### 函数原型

```lua
function md5:hexdigest()
end
```

#### 返回值

返回32个字符的十六进制字符串（大写）

### copy

复制哈希对象及其状态

#### 函数原型

```lua
function md5:copy()
end
```

#### 返回值

返回包含相同内部状态的新MD5对象

## 常见用途

### 文件校验

```lua
local md5 = require 'hashings.md5'

local function file_md5(filename)
    local file = io.open(filename, 'rb')
    if not file then return nil end

    local h = md5:new()
    while true do
        local block = file:read(64 * 1024)  -- 64KB块
        if not block then break end
        h:update(block)
    end
    file:close()
    return h:hexdigest()
end

print(file_md5('test.txt'))
```

### 快速哈希表

```lua
local md5 = require 'hashings.md5'

local function hash_key(key)
    local h = md5:new(key)
    -- 使用前8个字节作为64位哈希值
    local digest = h:digest()
    local hash = 0
    for i = 1, 8 do
        hash = hash * 256 + string.byte(digest, i)
    end
    return hash
end
```

## 性能特点

- **计算速度**：非常快
- **内存占用**：低（64字节缓冲区）
- **适用场景**：需要速度但不需要安全性的场景

## 类属性

```lua
md5.digest_size  -- 16 (字节)
md5.block_size   -- 64 (字节)
```

## 替代方案

对于需要安全性的场景，请使用：
- **SHA-256**：通用安全哈希
- **BLAKE2b**：高性能且安全
- **PBKDF2-HMAC-SHA256**：密码存储
