---

# SHA-1哈希算法 (hashings.sha1)

SHA-1（Secure Hash Algorithm 1）产生160位（20字节）的哈希值。

⚠️ **安全警告**：SHA-1已被证明存在碰撞攻击（SHAttered攻击），**不应再用于安全目的**。仅适用于非安全场景的数据完整性校验。

## 特性

- **摘要长度**：160位（20字节）
- **块大小**：512位（64字节）
- **输出格式**：通常表示为40个十六进制字符

## 使用方法

```lua
local sha1 = require 'hashings.sha1'

-- 创建SHA-1哈希对象并计算哈希
local h = sha1:new('Hello World')
print(h:hexdigest())  -- 输出: 0A0A9F2A6772942557AB5355C76F4B4F2A9B4F4

-- 流式处理
local h = sha1:new()
h:update('First part ')
h:update('Second part')
print(h:hexdigest())

-- 获取二进制输出
local h = sha1:new('Hello World')
local binary = h:digest()  -- 20字节二进制数据
```

## 接口说明

所有哈希算法都支持统一的接口：

### new

创建SHA-1哈希对象

#### 函数原型

```lua
function sha1:new(data)
end
```

#### 参数说明

* data
  可选，初始数据字符串

#### 返回值

返回SHA-1哈希对象

### update

更新哈希计算状态

#### 函数原型

```lua
function sha1:update(data)
end
```

#### 参数说明

* data
  要添加到哈希计算中的数据字符串

### digest

获取二进制哈希摘要

#### 函数原型

```lua
function sha1:digest()
end
```

#### 返回值

返回20字节的二进制字符串

### hexdigest

获取十六进制哈希摘要

#### 函数原型

```lua
function sha1:hexdigest()
end
```

#### 返回值

返回40个字符的十六进制字符串（大写）

### copy

复制哈希对象及其状态

#### 函数原型

```lua
function sha1:copy()
end
```

#### 返回值

返回包含相同内部状态的新SHA-1对象

## 使用示例

### Git提交校验

```lua
local sha1 = require 'hashings.sha1'

local function git_blob_hash(content)
    local h = sha1:new()
    h:update('blob ' .. #content .. '\0')
    h:update(content)
    return h:hexdigest()
end

-- 计算Git blob哈希
local content = 'Hello World'
local git_hash = git_blob_hash(content)
print('Git hash:', git_hash)
```

### 文件指纹

```lua
local sha1 = require 'hashings.sha1'

local function file_fingerprint(filename)
    local file = io.open(filename, 'rb')
    if not file then return nil end

    local h = sha1:new()
    while true do
        local block = file:read(64 * 1024)
        if not block then break end
        h:update(block)
    end
    file:close()

    return h:hexdigest()
end

local fingerprint = file_fingerprint('document.txt')
print('SHA-1:', fingerprint)
```

### 简单哈希表

```lua
local sha1 = require 'hashings.sha1'

local function hash_key(key)
    local h = sha1:new(key)
    local digest = h:digest()

    -- 使用前4个字节作为32位哈希值
    local hash = 0
    for i = 1, 4 do
        hash = hash * 256 + string.byte(digest, i)
    end

    return hash
end
```

## 性能特点

- **计算速度**：快，但比MD5慢
- **内存占用**：低（64字节缓冲区）
- **适用场景**：需要速度但不需要安全性的场景

## 类属性

```lua
sha1.digest_size  -- 20 (字节)
sha1.block_size   -- 64 (字节)
```

## 安全注意事项

⚠️ **重要**：

- **不安全**：自2017年起，SHA-1已被证明存在实际碰撞攻击
- **已被弃用**：许多组织已停止使用SHA-1用于数字签名
- **兼容性**：仍用于一些旧系统的向后兼容

## 替代方案

对于需要安全性的场景，请使用：
- **SHA-256**：通用安全哈希（推荐）
- **SHA-512**：更高安全性
- **BLAKE2系列**：高性能且安全

## 迁移指南

### 从SHA-1迁移到SHA-256

```lua
-- 之前（不安全）
local sha1 = require 'hashings.sha1'
local h = sha1:new(data)
local hash = h:hexdigest()  -- 40字符

-- 之后（安全）
local sha256 = require 'hashings.sha256'
local h = sha256:new(data)
local hash = h:hexdigest()  -- 64字符
```

注意事项：
- SHA-256摘要更长（64字符 vs 40字符）
- SHA-256计算稍慢（但更安全）
- 存储空间需求增加60%
