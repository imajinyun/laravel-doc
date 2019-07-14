# 数据库起步

## 简介

Laravel 使用原始 SQL、[流查询构建器](https://laravel.com/docs/5.8/queries) 和 [Eloquent ORM](https://laravel.com/docs/5.8/eloquent)，使与跨各种数据库后端的数据库交互极其地简单。目前，Laravel 支持四个数据库：

* MySQL
* PostgreSQL
* SQLite
* SQL Server

### 配置

你的应用程序的数据库配置位于 `config/database.php`。在此文件中，你可以定义所有数据库连接，以及指定默认情况下应使用的连接。此文件中提供了大多数受支持的数据库系统的示例。

默认情况下，Laravel 的样例 [环境配置](https://laravel.com/docs/5.8/configuration#environment-configuration) 可以与 [Laravel Homestead](https://laravel.com/docs/5.8/homestead) 一起使用，Homestead 是在你的本地机器上进行 Laravel 开发的一个方便的虚拟机。你可以根据你的本地数据库的需要修改此配置。

**SQLite 配置**

使用 `touch database/database.sqlite` 等命令创建新的 SQLite 数据库之后，你可以使用数据库的绝对路径轻松配置你的环境变量以指向此新创建的数据库：

```ini
DB_CONNECTION=sqlite
DB_DATABASE=/absolute/path/to/database.sqlite
```

要为 SQLite 连接启用外键约束，你应将 `foreign_key_constraints` 选项添加到你的 `config/database.php` 配置文件中：

```php
'sqlite' => [
    // ...
    'foreign_key_constraints' => true,
],
```

**使用 URL 配置**

通常，数据库连接使用多个配置值进行配置，如：`host`、`database`、`username`、`password` 等。每个配置值都有自己对应的环境变量。这意味着在生产服务器上配置你的数据库连接信息时，需要管理几个环境变量。

一些托管数据库提供者（如：Heroku）提供一个单一的数据库『URL』，其中的一个字符串中包含数据库的所有连接信息。示例数据库 URL 可能如下所示：

```bash
mysql://root:password@127.0.0.1/forge?charset=UTF-8
```

这些 URL 通常遵循标准模式约定：

```bash
driver://username:password@host:port/database?options
```

为了方便起见，Laravel 支持使用这些 URL 来替代使用多个配置选项配置数据库。如果 `url`（或对应的 `DATABASE_URL` 环境变量）配置选项存在，则将使用它提取数据库连接和凭据信息。

### 读 & 写连接

有时，您可能希望为 SELECT 语句使用一个数据库连接，为 INSERT、UPDATE 和 DELETE 语句使用另一个数据库连接。Laravel 使这变得很容易，并且无论你是使用原始查询、查询生成器还是 Eloquent ORM，都将始终使用适当的连接。

要了解应如何配置读 / 写连接，让我们看一下这个例子：

```php
'mysql' => [
    'read' => [
        'host' => [
            '192.168.1.1',
            '196.168.1.2',
        ],
    ],
    'write' => [
        'host' => [
            '196.168.1.3',
         ],
    ],
    'sticky'    => true,
    'driver'    => 'mysql',
    'database'  => 'database',
    'username'  => 'root',
    'password'  => '',
    'charset'   => 'utf8mb4',
    'collation' => 'utf8mb4_unicode_ci',
    'prefix'    => '',
],
```

#### `sticky` 选项

### 使用多个数据库连接

## 运行原始 SQL 查询

## 监听查询事件

## 数据库事务
