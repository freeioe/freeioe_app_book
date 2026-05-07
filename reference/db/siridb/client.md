---

# SiriDB客户端模块 (db.siridb.client)

本模块提供SiriDB数据库管理的HTTP客户端接口，用于执行数据库和用户管理操作。

## 功能说明

SiriDB客户端模块提供了完整的数据库管理功能：

- **数据库管理**：创建、删除数据库
- **用户管理**：创建账户、修改密码、删除账户
- **连接池管理**：创建连接池、管理副本
- **服务器信息**：获取版本、账户列表、数据库列表

本模块使用SiriDB的HTTP API进行管理操作，与database模块（数据操作）配合使用。

## 使用方法

```lua
local client = require 'db.siridb.client'

-- 创建客户端
local cli = client:new({
    host = '127.0.0.1',
    port = 9020,
    username = 'sa',
    password = 'siri',
    ssl = false
})

-- 创建新数据库
local ok, err = cli:new_database('mydb', 'ms', 1024)

-- 获取所有数据库
local databases = cli:get_databases()
```

## 接口说明

### new (initialize)

创建客户端对象

#### 函数原型

```lua
function client:initialize(options)
end
```

#### 参数说明

* options
  配置选项表

| 选项 | 类型 | 默认值 | 说明 |
| :--- | :--- | :--- | :--- |
| host | string | '127.0.0.1' | SiriDB服务器地址 |
| port | number | 9020 | HTTP端口 |
| ssl | boolean | false | 是否使用SSL/TLS |
| username | string | 'sa' | 用户名 |
| password | string | 'siri' | 密码 |
| timeout | number | nil | 请求超时（毫秒） |

#### 返回值

返回客户端对象

#### 示例

```lua
local client = require 'db.siridb.client'

-- 使用默认认证
local cli1 = client:new()
cli1:initialize({
    host = 'localhost',
    port = 9020
})

-- 使用自定义认证
local cli2 = client:new()
cli2:initialize({
    host = 'siridb.example.com',
    port = 9020,
    username = 'admin',
    password = 'secret_password',
    ssl = true,
    timeout = 10000
})
```

### new_database

创建新数据库

#### 函数原型

```lua
function client:new_database(dbname, time_precision, buffer_size, duration_num, duration_log)
end
```

#### 参数说明

* dbname
  数据库名称（必需）
* time_precision
  时间精度（可选，默认'ms'）
* buffer_size
  缓冲区大小（可选，默认1024）
* duration_num
  持续时间数量（可选）
* duration_log
  持续时间日志（可选）

#### 返回值

成功返回true，失败返回nil和错误信息

#### 时间精度选项

- **'s'**：秒
- **'ms'**：毫秒
- **'us'**：微秒
- **'ns'**：纳秒

#### 示例

```lua
-- 创建基本数据库
local ok, err = cli:new_database('mydb')

-- 创建带精度的数据库
local ok, err = cli:new_database('sensors', 'ms')

-- 创建带完整配置的数据库
local ok, err = cli:new_database('production', 'ms', 2048, 365, '1d')

if not ok then
    print('Failed to create database:', err)
end
```

### drop_database

删除数据库

#### 函数原型

```lua
function client:drop_database(dbname, ignore_offline)
end
```

#### 参数说明

* dbname
  数据库名称（必需）
* ignore_offline
  是否忽略离线状态（可选，默认false）

#### 返回值

成功返回响应数据，失败返回nil和错误信息

#### 示例

```lua
-- 删除数据库
local ok, err = cli:drop_database('old_db')

-- 强制删除（忽略离线状态）
local ok, err = cli:drop_database('old_db', true)

if not ok then
    print('Failed to drop database:', err)
end
```

### new_account

创建新账户

#### 函数原型

```lua
function client:new_account(user, passwd)
end
```

#### 参数说明

* user
  用户名（必需）
* passwd
  密码（必需）

#### 返回值

成功返回响应数据，失败返回nil和错误信息

#### 示例

```lua
-- 创建新用户
local result, err = cli:new_account('john', 'secret_password')

if result then
    print('Account created successfully')
else
    print('Failed to create account:', err)
end
```

### change_password

修改账户密码

#### 函数原型

```lua
function client:change_password(user, password)
end
```

#### 参数说明

* user
  用户名（必需）
* password
  新密码（必需）

#### 返回值

成功返回响应数据，失败返回nil和错误信息

#### 示例

```lua
-- 修改密码
local result, err = cli:change_password('john', 'new_password')

if not result then
    print('Failed to change password:', err)
end
```

### drop_account

删除账户

#### 函数原型

```lua
function client:drop_account(user)
end
```

#### 参数说明

* user
  用户名（必需）

#### 返回值

成功返回响应数据，失败返回nil和错误信息

#### 示例

```lua
-- 删除账户
local result, err = cli:drop_account('john')

if result then
    print('Account deleted successfully')
end
```

### new_pool

创建连接池

#### 函数原型

```lua
function client:new_pool(dbname, user, passwd, host, port)
end
```

#### 参数说明

* dbname
  数据库名称（必需）
* user
  用户名（可选）
* passwd
  密码（可选）
* host
  主机地址（可选）
* port
  端口号（可选）

#### 返回值

成功返回响应数据，失败返回nil和错误信息

#### 示例

```lua
-- 创建连接池
local result, err = cli:new_pool('mydb', 'iris', 'siri', 'localhost', 9000)

if result then
    print('Pool created successfully')
end
```

### new_replica

创建副本

#### 函数原型

```lua
function client:new_replica(dbname, user, passwd, host, port, pool)
end
```

#### 参数说明

* dbname
  数据库名称（必需）
* user
  用户名（可选）
* passwd
  密码（可选）
* host
  主机地址（可选）
* port
  端口号（可选）
* pool
  连接池名称（可选）

#### 返回值

成功返回响应数据，失败返回nil和错误信息

#### 示例

```lua
-- 创建副本
local result, err = cli:new_replica(
    'mydb',
    'iris',
    'siri',
    'replica.example.com',
    9000,
    'my_pool'
)
```

### get_version

获取服务器版本

#### 函数原型

```lua
function client:get_version()
end
```

#### 返回值

成功返回版本号字符串，失败返回nil和错误信息

#### 示例

```lua
-- 获取版本
local version, err = cli:get_version()
if version then
    print('SiriDB version:', version)
else
    print('Failed to get version:', err)
end
```

### get_accounts

获取所有账户

#### 函数原型

```lua
function client:get_accounts()
end
```

#### 返回值

成功返回账户列表，失败返回nil

#### 示例

```lua
-- 获取账户列表
local accounts = cli:get_accounts()
if accounts then
    print('Accounts:')
    for _, account in ipairs(accounts) do
        print('  -', account)
    end
end
```

### get_databases

获取所有数据库

#### 函数原型

```lua
function client:get_databases()
end
```

#### 返回值

成功返回数据库列表，失败返回nil

#### 示例

```lua
-- 获取数据库列表
local databases = cli:get_databases()
if databases then
    print('Databases:')
    for _, db in ipairs(databases) do
        print('  -', db)
    end
end
```

### post

发送POST请求（底层方法）

#### 函数原型

```lua
function client:post(url, params, data)
end
```

#### 参数说明

* url
  请求URL路径
* params
  查询参数（可选）
* data
  请求体数据（可选）

#### 返回值

成功返回响应数据（JSON解码）和响应体，失败返回nil和响应体

#### 示例

```lua
-- 自定义POST请求
local result, body = cli:post('/custom-endpoint', nil, {
    key1 = 'value1',
    key2 = 'value2'
})
```

### get

发送GET请求（底层方法）

#### 函数原型

```lua
function client:get(url, params, data)
end
```

#### 参数说明

* url
  请求URL路径
* params
  查询参数（可选）
* data
  请求数据（可选）

#### 返回值

成功返回响应数据（JSON解码），失败返回nil和错误信息

#### 示例

```lua
-- 自定义GET请求
local result, err = cli:get('/info', nil, nil)
if result then
    print('Server info:', result)
end
```

## 完整示例

### 完整的数据库设置流程

```lua
local client = require 'db.siridb.client'

-- 连接到SiriDB服务器
local cli = client:new()
cli:initialize({
    host = 'localhost',
    port = 9020,
    username = 'sa',
    password = 'siri'
})

-- 1. 检查服务器版本
local version, err = cli:get_version()
if version then
    print('Connected to SiriDB version:', version)
else
    print('Failed to get version:', err)
    return
end

-- 2. 创建新数据库
local ok, err = cli:new_database('production', 'ms', 2048)
if not ok then
    print('Failed to create database:', err)
    return
end
print('Database created successfully')

-- 3. 创建用户账户
local result, err = cli:new_account('app_user', 'secure_password')
if result then
    print('User account created')
end

-- 4. 创建连接池
local result, err = cli:new_pool('production', 'app_user', 'secure_password', 'localhost', 9000)
if result then
    print('Connection pool created')
end

-- 5. 验证设置
local databases = cli:get_databases()
print('Available databases:', table.concat(databases, ', '))
```

### 用户管理

```lua
local client = require 'db.siridb.client'

local cli = client:new()
cli:initialize({
    host = 'localhost',
    port = 9020
})

-- 创建多个用户
local users = {
    {name='readonly', password='ro_pass'},
    {name='readwrite', password='rw_pass'},
    {name='admin', password='admin_pass'}
}

for _, user in ipairs(users) do
    local result, err = cli:new_account(user.name, user.password)
    if result then
        print(string.format('User %s created', user.name))
    else
        print(string.format('Failed to create user %s: %s', user.name, err))
    end
end

-- 列出所有用户
local accounts = cli:get_accounts()
print('All accounts:', table.concat(accounts, ', '))

-- 修改用户密码
local result, err = cli:change_password('readonly', 'new_password')
if result then
    print('Password changed successfully')
end

-- 删除用户
local result, err = cli:drop_account('admin')
if result then
    print('User deleted successfully')
end
```

### 数据库生命周期管理

```lua
local client = require 'db.siridb.client'

local cli = client:new()
cli:initialize({
    host = 'localhost',
    port = 9020
})

-- 创建开发数据库
local function setup_dev_environment()
    local db_name = 'dev_' .. os.date('%Y%m%d')

    -- 创建数据库
    local ok, err = cli:new_database(db_name, 'ms', 512)
    if not ok then
        print('Failed to create dev database:', err)
        return nil
    end

    -- 创建测试用户
    local result, err = cli:new_account('tester', 'test_pass')
    if not result then
        print('Failed to create tester account:', err)
    end

    print('Dev environment setup complete:', db_name)
    return db_name
end

-- 清理旧数据库
local function cleanup_old_databases(days_old)
    local cutoff = os.time() - (days_old * 86400)

    local databases = cli:get_databases()
    if not databases then
        return
    end

    for _, db in ipairs(databases) do
        -- 检查是否是开发数据库
        local dev_prefix = db:match('^dev_(%d+)')
        if dev_prefix then
            local db_date = tonumber(dev_prefix)
            if db_date and db_date < cutoff then
                print('Removing old database:', db)
                cli:drop_database(db, true)
            end
        end
    end
end

-- 使用
setup_dev_environment()
cleanup_old_databases(7)  -- 清理7天前的开发数据库
```

### 集群配置

```lua
local client = require 'db.siridb.client'

-- 主服务器配置
local primary = client:new()
primary:initialize({
    host = 'siridb-primary.example.com',
    port = 9020,
    username = 'sa',
    password = 'siri'
})

-- 创建数据库
local ok, err = primary:new_database('cluster_db', 'ms', 2048)
if not ok then
    print('Failed to create database:', err)
    return
end

-- 创建连接池
local ok, err = primary:new_pool('cluster_db', 'replica_user', 'replica_pass', 'primary.example.com', 9000)

-- 副本服务器配置
local replica_servers = {
    {host='replica1.example.com', port=9000},
    {host='replica2.example.com', port=9000}
}

for i, server in ipairs(replica_servers) do
    local replica = client:new()
    replica:initialize({
        host = server.host,
        port = 9020,
        username = 'sa',
        password = 'siri'
    })

    -- 添加副本
    local result, err = replica:new_replica(
        'cluster_db',
        'replica_user',
        'replica_pass',
        server.host,
        9000,
        'cluster_pool'
    )

    if result then
        print(string.format('Replica %d created successfully', i))
    else
        print(string.format('Failed to create replica %d: %s', i, err))
    end
end
```

### 错误处理和重试

```lua
local client = require 'db.siridb.client'

-- 带重试的数据库创建
local function create_database_with_retry(cli, dbname, max_retries)
    max_retries = max_retries or 3

    for attempt = 1, max_retries do
        local ok, err = cli:new_database(dbname, 'ms', 1024)

        if ok then
            print(string.format('Database %s created on attempt %d', dbname, attempt))
            return true
        end

        print(string.format('Attempt %d failed: %s', attempt, err or 'unknown error'))

        if attempt < max_retries then
            print('Retrying in 2 seconds...')
            os.execute('sleep 2')
        end
    end

    print(string.format('Failed to create database after %d attempts', max_retries))
    return false
end

-- 使用
local cli = client:new()
cli:initialize({host='localhost', port=9020})

create_database_with_retry(cli, 'test_db', 5)
```

### 批量操作

```lua
local client = require 'db.siridb.client'

-- 批量创建数据库
local function create_multiple_databases(cli, db_list)
    local results = {}

    for _, db_info in ipairs(db_list) do
        local ok, err = cli:new_database(
            db_info.name,
            db_info.precision or 'ms',
            db_info.buffer_size or 1024
        )

        results[db_info.name] = {
            success = ok,
            error = err
        }

        if ok then
            print(string.format('✓ Created database: %s', db_info.name))
        else
            print(string.format('✗ Failed to create %s: %s', db_info.name, err))
        end
    end

    return results
end

-- 使用
local cli = client:new()
cli:initialize({host='localhost', port=9020})

local databases = {
    {name='sensors', precision='ms', buffer_size=2048},
    {name='logs', precision='s', buffer_size=4096},
    {name='metrics', precision='ms', buffer_size=1024}
}

create_multiple_databases(cli, databases)
```

## 最佳实践

1. **错误处理**：始终检查返回值并处理错误
2. **资源清理**：定期清理不再使用的数据库
3. **密码安全**：使用强密码并妥善保管
4. **连接管理**：合理配置连接池大小
5. **权限控制**：根据需要创建不同权限的用户
6. **监控**：定期检查服务器状态和版本

## 注意事项

1. **权限要求**：管理操作需要管理员权限
2. **数据库删除**：删除操作不可逆，请谨慎操作
3. **密码格式**：密码应符合复杂度要求
4. **连接限制**：注意服务器的连接数限制
5. **时间精度**：数据库创建后无法修改时间精度

## 错误排查

### 常见错误及解决方法

```lua
-- 1. 认证失败
-- 错误：Invalid credentials
-- 解决：检查username和password是否正确

-- 2. 数据库已存在
-- 错误：Database already exists
-- 解决：使用不同的数据库名或先删除现有数据库

-- 3. 权限不足
-- 错误：Permission denied
-- 解决：使用管理员账户（通常是sa）

-- 4. 连接超时
-- 错误：Connection timeout
-- 解决：检查host和port，设置合适的timeout

-- 5. 无效的配置
-- 错误：Invalid configuration
-- 解决：检查参数类型和值是否有效
```

## 与database模块配合

```lua
local client = require 'db.siridb.client'
local database = require 'db.siridb.database'

-- 1. 使用client创建数据库
local cli = client:new()
cli:initialize({host='localhost', port=9020})

cli:new_database('myapp', 'ms', 1024)

-- 2. 使用database连接到数据库
local db = database:new({
    host = 'localhost',
    port = 9000,
    username = 'iris',
    password = 'siri',
    time_precision = 'ms'
}, 'myapp')

-- 3. 执行数据操作
local sdata = require 'db.siridb.data'
local sseries = require 'db.siridb.series'

local series = sseries:new('temperature', 'float')
series:push_value(25.5, skynet.time())

local data = sdata:new()
data:add_series('temperature', series)

db:insert(data, true)
```

## API端点参考

### 可用的HTTP端点

| 端点 | 方法 | 功能 |
| :--- | :--- | :--- |
| /new-database | POST | 创建新数据库 |
| /drop-database | POST | 删除数据库 |
| /new-account | POST | 创建新账户 |
| /change-password | POST | 修改密码 |
| /drop-account | POST | 删除账户 |
| /new-pool | POST | 创建连接池 |
| /get-version | GET | 获取服务器版本 |
| /get-accounts | GET | 获取账户列表 |
| /get-databases | GET | 获取数据库列表 |
