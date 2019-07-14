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

注意，已经向配置数组添加了三个键：`read`、`write` 和 `sticky`。`read` 和 `write` 键的数组值包含一个键：`host`。`read` 和 `write` 连接的其他数据库选项将从主 `mysql` 数组中合并。

如果你希望从主数组中覆盖值，仅需要你将条目放在 `read` 和 `write` 数组中。因此，在本例中，`192.168.1.1` 将用作『读』连接的主机，而 `192.168.1.3` 将用作『写』连接。主 `mysql` 数组中的数据库凭据、前缀、字符集和所有其他选项将在两个连接之间共享。

#### `sticky` 选项

`sticky` 选项是一个可选值，可用于允许立即读取在当前请求周期中写入数据库的记录。如果启用了 `sticky` 选项，并且在当前请求周期中对数据库执行了『写』操作，那么任何进一步的『读』操作都将使用『写』连接。这可以确保在请求周期中编写的任何数据都可以在相同的请求期间立即从数据库中读取出来。你可以决定这是否是你的应用程序期望的行为。

### 使用多个数据库连接

使用多个连接时，你可以通过 `DB` 外观上的 `connection` 方法访问每个连接。传递给 `connection` 方法的 `name` 应该对应于在你的 `config/database.php` 配置文件中列出的连接之一：

```php
$users = DB::connection('foo')->select(...);
```

你还可以使用连接实例上的 `getPdo` 方法访问原始的底层 PDO 实例：

```php
$pdo = DB::connection()->getPdo();
```

## 运行原始 SQL 查询

一旦你配置了你的数据库连接，你就可以使用 `DB` 外观运行查询。`DB` 外观为每种查询类型提供了方法：`select`、`update`、`insert`、`delete` 和 `statement`。

### 运行 Select 查询

要运行基本查询，你可以在 `DB` 外观上使用 `select` 方法：

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Support\Facades\DB;
use App\Http\Controllers\Controller;

class UserController extends Controller
{
    /**
     * 显示应用程序所有用户的列表。
     *
     * @return Response
     */
    public function index()
    {
        $users = DB::select('select * from users where active = ?', [1]);

        return view('user.index', ['users' => $users]);
    }
}
```

传递给 `select` 方法的第一个参数是原始 SQL 查询，第二个参数是需要绑定到查询的任何参数绑定。通常，这些是 `where` 子句约束的值。参数绑定提供了对 SQL 注入的保护。

## 监听查询事件

## 数据库事务
