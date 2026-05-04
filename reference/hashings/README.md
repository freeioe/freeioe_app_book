---

# 哈希模块集合 (hashings)

本模块提供多种哈希算法和密钥派生函数的实现，基于 [lua-hashings](https://github.com/user-none/lua-hashings)。

## 模块列表

### 加密哈希算法

| 模块 | 摘要长度 | 说明 |
| :--- | :--- | :--- |
| [md5](md5.md) | 128位 | MD5哈希算法（已不推荐用于安全用途） |
| [sha1](sha1.md) | 160位 | SHA-1哈希算法（已不推荐用于安全用途） |
| [sha256](sha256.md) | 256位 | SHA-256哈希算法（SHA-2家族） |
| [sha512](sha512.md) | 512位 | SHA-512哈希算法（SHA-2家族） |
| [sha3_256](sha3_256.md) | 256位 | SHA3-256哈希算法（SHA-3家族） |
| [sha3_512](sha3_512.md) | 512位 | SHA3-512哈希算法（SHA-3家族） |
| [blake2b](blake2b.md) | 可变 | BLAKE2b哈希算法（速度快，安全性高） |
| [blake2s](blake2s.md) | 可变 | BLAKE2s哈希算法（32位优化） |
| [ripemd160](ripemd160.md) | 160位 | RIPEMD-160哈希算法 |
| [whirlpool](whirlpool.md) | 512位 | Whirlpool哈希算法 |

### 校验和算法

| 模块 | 长度 | 说明 |
| :--- | :--- | :--- |
| [crc32](crc32.md) | 32位 | CRC32校验和（广泛用于文件校验） |
| [adler32](adler32.md) | 32位 | Adler32校验和（速度快于CRC32） |
| [fcs16](fcs16.md) | 16位 | FCS16校验和 |
| [fcs32](fcs32.md) | 32位 | FCS32校验和 |
| [sum16](sum16.md) | 16位 | 简单16位求和校验 |
| [sum](sum.md) | 8位 | 简单8位求和校验 |

### 密钥派生和消息认证

| 模块 | 说明 |
| :--- | :--- |
| [hmac](hmac.md) | HMAC消息认证码 |
| [pbkdf2](pbkdf2.md) | PBKDF2密钥派生函数 |

## 使用方法

### 基本用法

```lua
local hashings = require 'hashings'

-- 方法1：使用统一接口
local md5 = hashings.md5
local h = md5:new('Hello World')
print(h:hexdigest())  -- 输出: B10A8DB164E0754105B7A99BE72E3FE5

-- 方法2：直接调用模块
local sha256 = require 'hashings.sha256'
local h = sha256:new('Hello World')
print(h:hexdigest())  -- 输出SHA-256哈希值
```

### 流式更新

```lua
local sha256 = require 'hashings.sha256'

-- 创建哈希对象
local h = sha256:new()

-- 分块更新数据
h:update('First part ')
h:update('Second part ')
h:update('Third part')

-- 获取最终哈希值
print(h:hexdigest())
```

### 二进制输出

```lua
local sha256 = require 'hashings.sha256'

local h = sha256:new('Hello World')
local binary_hash = h:digest()  -- 返回二进制字符串
local hex_hash = h:hexdigest()   -- 返回十六进制字符串
```

### 复制哈希状态

```lua
local sha256 = require 'hashings.sha256'

local h1 = sha256:new('Hello')
local h2 = h1:copy()  -- 复制当前状态

h1:update(' World')
h2:update(' There')

print(h1:hexdigest())  -- "Hello World"的哈希
print(h2:hexdigest())  -- "Hello There"的哈希
```

### 使用HMAC

```lua
local hashings = require 'hashings'

-- 创建HMAC-SHA256
local hmac = hashings.hmac
local sha256 = hashings.sha256

local h = hmac:new(sha256, 'secret_key', 'message')
print(h:hexdigest())
```

### 使用PBKDF2

```lua
local hashings = require 'hashings'

-- 派生密钥（用于密码哈希）
local pbkdf2 = hashings.pbkdf2
local sha256 = hashings.sha256

local derived_key = pbkdf2:pbkdf2(sha256, 'password', 'salt', 10000)
print(derived_key)  -- 输出派生密钥的十六进制表示
```

## 统一接口

所有哈希算法都支持以下接口：

### 类属性

* digest_size
  哈希摘要的字节长度
* block_size
  内部处理块的字节长度

### 方法

#### new

创建哈希对象

```lua
local hash_obj = algorithm:new(data)
```

**参数：**
- data（可选）- 初始数据

**返回：**
- 哈希对象

#### update

更新哈希状态

```lua
hash_obj:update(data)
```

**参数：**
- data - 要添加的数据

#### digest

获取二进制哈希摘要

```lua
local binary_hash = hash_obj:digest()
```

**返回：**
- 二进制字符串

#### hexdigest

获取十六进制哈希摘要

```lua
local hex_hash = hash_obj:hexdigest()
```

**返回：**
- 十六进制编码的字符串

#### copy

复制哈希对象（包括当前状态）

```lua
local copy = hash_obj:copy()
```

**返回：**
- 新的哈希对象，包含相同的内部状态

## 性能建议

1. **选择合适的算法**：
   - MD5/SHA1：仅用于非安全目的的校验和
   - SHA-256：通用安全哈希，性能与安全性平衡
   - BLAKE2系列：性能最优，安全性高，推荐用于新项目
   - SHA-512：需要更长摘要时使用

2. **大数据处理**：
   - 使用流式更新（`update`）而非一次性处理大数据
   - 避免频繁的`digest`调用，仅在需要时调用

3. **密码存储**：
   - 不要直接使用MD5/SHA存储密码
   - 使用PBKDF2、bcrypt或scrypt等专门的密码哈希算法
   - 设置足够的迭代次数（建议10000+）

## 安全注意事项

⚠️ **重要安全提示**：

- **MD5和SHA-1**已被证明存在碰撞攻击，不应再用于安全目的
- **密钥管理**：使用HMAC时确保密钥的安全存储
- **盐值使用**：密码哈希必须使用随机盐值
- **迭代次数**：PBKDF2的迭代次数应根据当前硬件性能设置

## 算法选择指南

| 用途 | 推荐算法 |
| :--- | :--- |
| 数据完整性校验 | CRC32, SHA-256 |
| 密码存储 | PBKDF2-SHA256（10000+迭代） |
| 消息认证 | HMAC-SHA256 |
| 数字签名 | SHA-256, SHA-3 |
| 哈希表 | MD5, FNV（快速但非安全） |
| 文件校验和 | MD5, SHA-256 |
| 新项目首选 | BLAKE2b, SHA-256 |
