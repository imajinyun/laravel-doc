# 数据库：迁移

## 简介

迁移类似于你的数据库的版本控制，允许你的团队轻松地修改和共享应用程序的数据库模式。迁移通常与 Laravel 的模式构建器配对，以便轻松地构建你的应用程序的数据库模式。如果你曾经不得不告诉一个团队成员手动地将一个列添加到他们的本地数据库模式中，那么你就遇到了数据库迁移所解决的问题。

Laravel `Schema` [外观](https://laravel.com/docs/5.8/facades) 为跨所有 Laravel 支持的数据库系统创建和操作表提供了与数据库无关的支持。

## 生成迁移

要创建一个迁移，使用 `make:migration` [Artisan 命令](https://laravel.com/docs/5.8/artisan)：

```bash
php artisan make:migration create_users_table
```

新迁移将放在你的 `database/migrations` 目录中。每个迁移文件名都包含一个时间戳，允许 Laravel 确定迁移的顺序。

`—-table` 和 `-—create` 选项还可以用来指示表的名称，以及迁移是否将创建一个新表。这些选项用指定的表预先填充生成的迁移存根文件：

```bash
php artisan make:migration create_users_table --create=users

php artisan make:migration add_votes_to_users_table --table=users
```

如果希望为生成的迁移指定自定义输出路径，你可以在执行 `make:migration` 命令时使用 `——path` 选项。给定的路径应该相对于你的应用程序的基本路径。

## 迁移结构

迁移类包含两个方法：`up` 和 `down`。`up` 方法用于向你的数据库添加新表、列或索引，而 `down` 方法应该反转 `up` 方法执行的操作。

在这两种方法中，你都可以使用 Laravel 模式构建器来表现性地创建和修改表。要了解 `Schema` 构建器上所有可用的方法，[查看其文档](https://laravel.com/docs/5.8/migrations#creating-tables)。例如，这个迁移示例创建了一个 `flights` 表：

```php
<?php

use Illuminate\Support\Facades\Schema;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class CreateFlightsTable extends Migration
{
    /**
     * 运行迁移。
     *
     * @return void
     */
    public function up()
    {
        Schema::create('flights', function (Blueprint $table) {
            $table->bigIncrements('id');
            $table->string('name');
            $table->string('airline');
            $table->timestamps();
        });
    }

    /**
     * 反转迁移。
     *
     * @return void
     */
    public function down()
    {
        Schema::drop('flights');
    }
}
```

## 运行迁移

要运行你的所有未完成的迁移，执行 `migrate` Artisan 命令：

```bash
php artisan migrate
```

{% hint style="danger" %}

如果您正在使用 [Homestead 虚拟机](https://laravel.com/docs/5.8/homestead)，则应该在你的虚拟机中运行此命令。

{% endhint  %}

**强制迁移在生产中运行**

有些迁移操作是破坏性的，这意味着它们可能导致你丢失数据。为了防止在你的生产数据库上运行这些命令，在执行这些命令之前将提示你进行确认。要强制命令在没有提示的情况下运行，请使用 `——force` 标志：

```bash
php artisan migrate --force
```

### 回滚迁移

要回滚最新的迁移操作，可以使用 `rollback` 命令。此命令回滚最后『一批』迁移，其中可能包含多个迁移文件：

```bash
php artisan migrate:rollback
```

通过向 `rollback` 命令提供 `step` 选项，你可以回滚有限数量的迁移。例如，下面的命令将回滚最近的五个迁移：

```bash
php artisan migrate:rollback --step=5
```

`migration:reset` 命令将回滚你的应用程序的所有迁移：

```bash
php artisan migrate:reset
```

#### 回滚 & 迁移单个命令

`migrate:refresh` 命令将回滚你的所有迁移，然后执行 `migrate` 命令。此命令可以有效地重新创建整个数据库：

```bash
php artisan migrate:refresh

// 刷新数据库并运行所有数据库填充...
php artisan migrate:refresh --seed
```

通过向 `refresh` 命令提供 `step` 选项，你可以回滚和重新迁移有限数量的迁移。例如，下面的命令将回滚并重新迁移最近的五次迁移：

```bash
php artisan migrate:fresh --step=5
```

#### 删除所有表 & 迁移

`migrate:fresh` 命令将从数据库中删除所有数据表，然后执行 `migrate` 命令：

```bash
php artisan migrate:fresh

php artisan migrate:fresh --seed
```

## 表

### 创建表

要创建一个新的数据库表，使用 `Schema` 外观上的 `create` 方法。`create` 方法接受两个参数。第一个是表的名称，第二个是一个 `Closure`，它接收一个 `Blueprint` 对象，这个对象可以用来定义新表：

```php
Schema::create('users', function (Blueprint $table) {
    $table->bigIncrements('id');
});
```

在创建表时，你可以使用模式构建器的任何 [列方法](https://laravel.com/docs/5.8/migrations#creating-columns) 来定义表的列。

#### 检查表 / 列的存在

你可以使用 `hasTable` 和 `hasColumn` 方法轻松检查表或列是否存在：

```php
if (Schema::hasTable('users')) {
    //
}

if (Schema::hasColumn('users', 'email')) {
    //
}
```

#### 数据库连接和表选项

如果你希望对你的非默认连接的数据库连接执行模式操作，使用 `connection` 方法：

```php
Schema::connection('foo')->create('users', function (Blueprint $table) {
    $table->bigIncrements('id');
});
```

你可以在模式构建器上使用以下命令来定义表的选项：

| 命令 | 描述 |
| --- | --- |
| `$table->engine = 'InnoDB';` | 指定表存储引擎（MySQL） |
| `$table->charset = 'utf8';` | 指定表的默认字符集（MySQL） |
| `$table->collation = 'utf8_unicode_ci';` | 指定表的默认排序规则（MySQL） |
| `$table->temporary();` | 创建一个临时表（SQL Server 除外） |

### 重命名 / 删除表

若要重命名现有数据库表，使用 `rename` 方法：

```php
Schema::rename($from, $to);
```

要删除现有表，可以使用 `drop` 或 `dropIfExists` 方法：

```php
Schema::drop('users');

Schema::dropIfExists('users');
```

**使用外键重命名表**

在重命名表之前，你应该验证表上的任何外键约束在你的迁移文件中都有显式名称，而不是让 Laravel 分配基于约定的名称。否则，外键约束名将引用旧表名。

## 列

## 索引
