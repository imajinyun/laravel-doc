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

| 命令                                     | 描述                              |
| ---------------------------------------- | --------------------------------- |
| `$table->engine = 'InnoDB';`             | 指定表存储引擎（MySQL）           |
| `$table->charset = 'utf8';`              | 指定表的默认字符集（MySQL）       |
| `$table->collation = 'utf8_unicode_ci';` | 指定表的默认排序规则（MySQL）     |
| `$table->temporary();`                   | 创建一个临时表（SQL Server 除外） |

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

### 创建列

`Schema` 外观上的 `table` 方法可用于更新现有的表。与 `create` 方法一样，`table` 方法接受两个参数：表的名称和一个 `Closure`，该闭包接收一个 `Blueprint` 实例，你可以使用该实例向表添加列：

```php
Schema::table('users', function (Blueprint $table) {
    $table->string('email');
});
```

#### 可用的列类型

模式构建器包含各种列类型，你可以在构建你的表时指定这些列类型：

| 命令                                                 | 描述                                                                 |
| ---------------------------------------------------- | -------------------------------------------------------------------- |
| `$table->bigIncrements('id');`                       | 自动递增 UNSIGNED BIGINT（主键）等效列                               |
| `$table->bigInteger('votes');`                       | BIGINT 等效列                                                        |
| `$table->binary('data');`                            | BLOB 等效列                                                          |
| `$table->boolean('confirmed');`                      | BOOLEAN 等效列                                                       |
| `$table->char('name', 100);`                         | 可选长度的 CHAR 等效列                                               |
| `$table->date('created_at');`                        | DATE 等效列                                                          |
| `$table->dateTime('created_at');`                    | DATETIME 等效列                                                      |
| `$table->dateTimeTz('created_at');`                  | DATETIME（带时区） 等效列                                            |
| `$table->decimal('amount', 8, 2);`                   | 有精度（总位数）和刻度（十进制数字）的 DECIMAL 等效列                |
| `$table->double('amount', 8, 2);`                    | 有精度（总位数）和刻度（十进制数字）的 DOUBLE 等效列                 |
| `$table->enum('level', ['easy', 'hard']);`           | 枚举等效列                                                           |
| `$table->float('amount', 8, 2);`                     | 有精度（总位数）和刻度（十进制数字）的 FLOAT 等效列                  |
| `$table->geometry('positions');`                     | GEOMETRY 等效列                                                      |
| `$table->geometryCollection('positions');`           | GEOMETRYCOLLECTION 等效列                                            |
| `$table->increments('id');`                          | 自动递增 UNSIGNED INTEGER（主键）等效列                              |
| `$table->integer('votes');`                          | INTEGER 等效列                                                       |
| `$table->ipAddress('visitor');`                      | IP 地址等效列                                                        |
| `$table->json('options');`                           | JSON 等效列                                                          |
| `$table->jsonb('options');`                          | JSONB 等效列                                                         |
| `$table->lineString('positions');`                   | LINESTRING 等效列                                                    |
| `$table->longText('description');`                   | LONGTEXT 等效列                                                      |
| `$table->macAddress('device');`                      | MAC 地址等效列                                                       |
| `$table->mediumIncrements('id');`                    | 自动递增 UNSIGNED MEDIUMINT（主键）等效列                            |
| `$table->mediumInteger('votes');`                    | MEDIUMINT 等效列                                                     |
| `$table->mediumText('description');`                 | MEDIUMTEXT 等效列                                                    |
| `$table->morphs('taggable');`                        | 添加 `taggable_id` UNSIGNED BIGINT 和 `taggable_type` VARCHAR 等效列 |
| `$table->multiLineString('positions');`              | MULTILINESTRING 等效列                                               |
| `$table->multiPoint('positions');`                   | MULTIPOINT 等效列                                                    |
| `$table->multiPolygon('positions');`                 | MULTIPOLYGON 等效列                                                  |
| `$table->nullableMorphs('taggable');`                | 添加 `morphs()` 版本的可空列                                         |
| `$table->nullableTimestamps();`                      | `timestamps()` 方法的别名                                            |
| `$table->point('position');`                         | POINT 等效列                                                         |
| `$table->polygon('positions');`                      | POLYGON 等效列                                                       |
| `$table->rememberToken();`                           | 添加一个可空的 `remember_token` VARCHAR(100) 等效列                  |
| `$table->set('flavors', ['strawberry', 'vanilla']);` | SET 等效列                                                           |
| `$table->smallIncrements('id');`                     | 自动递增 UNSIGNED SMALLINT（主键）等效列                             |
| `$table->smallInteger('votes');`                     | SMALLINT 等效列                                                      |
| `$table->softDeletes();`                             | 为软删除添加一个可空的 `deleted_at` TIMESTAMP 等效列                 |
| `$table->softDeletesTz();`                           | 为软删除添加一个可空的 `deleted_at` TIMESTAMP（带时区）等效列        |
| `$table->string('name', 100);`                       | 可选长度的 VARCHAR 等效列                                            |
| `$table->text('description');`                       | TEXT 等效列                                                          |
| `$table->time('sunrise');`                           | TIME 等效列                                                          |
| `$table->timeTz('sunrise');`                         | TIME（带时区）等效列                                                 |
| `$table->timestamp('added_on');`                     | TIMESTAMP 等效列                                                     |
| `$table->timestampTz('added_on');`                   | TIMESTAMP（带时区）等效列                                            |
| `$table->timestamps();`                              | 添加可空的 `created_at` 和 `updated_at` TIMESTAMP 等效列             |
| `$table->timestampsTz();`                            | 添加可空的 `created_at` 和 `updated_at` TIMESTAMP（带时区）等效列    |
| `$table->tinyIncrements('id');`                      | 自动递增 UNSIGNED TINYINT（主键）等效列                              |
| `$table->tinyInteger('votes');`                      | TINYINT 等效列                                                       |
| `$table->unsignedBigInteger('votes');`               | UNSIGNED BIGINT 等效列                                               |
| `$table->unsignedDecimal('amount', 8, 2);`           | 有精度（总位数）和刻度（十进制数字）的 UNSIGNED DECIMAL 等效列       |
| `$table->unsignedInteger('votes');`                  | UNSIGNED INTEGER 等效列                                              |
| `$table->unsignedMediumInteger('votes');`            | UNSIGNED MEDIUMINT 等效列                                            |
| `$table->unsignedSmallInteger('votes');`             | UNSIGNED SMALLINT 等效列                                             |
| `$table->unsignedTinyInteger('votes');`              | UNSIGNED TINYINT 等效列                                              |
| `$table->uuid('id');`                                | UUID 等效列                                                          |
| `$table->year('birth_year');`                        | YEAR 等效列                                                          |

### 列修饰符

除了上面列出的列类型之外，向数据库表添加列的同时还可以使用几个列『修饰符』。例如，要使列『nullable』，可以使用 `nullable` 方法：

```php
Schema::table('users', function (Blueprint $table) {
    $table->string('email')->nullable();
});
```

下面是所有可用列修饰符的列表。此列表不包含 [索引修饰符](https://laravel.com/docs/5.8/migrations#creating-indexes)：

| 修饰符                           | 描述                                                   |
| -------------------------------- | ------------------------------------------------------ |
| `->after('column')`              | 在另一个列『之后』放置列（MySQL）                      |
| `->autoIncrement()`              | 设置 INTEGER 列为自动递增（主键）                      |
| `->charset('utf8')`              | 指定列的字符集（MySQL）                                |
| `->collation('utf8_unicode_ci')` | 指定列的排序规则（MySQL/PostgreSQL/SQL Server）        |
| `->comment('my comment')`        | 添加注释到一个列（MySQL/PostgreSQL）                   |
| `->default($value)`              | 为列指定一个『默认』值                                 |
| `->first()`                      | 将列放置在表的『第一』列上（MySQL）                    |
| `->nullable($value = true)`      | 允许（默认情况下）NULL 值插入到列中                    |
| `->storedAs($expression)`        | 创建存储生成的列（MySQL）                              |
| `->unsigned()`                   | 设置 INTEGER 列为 UNSIGNED（MySQL）                    |
| `->useCurrent()`                 | 设置 TIMESTAMP 列以使用 CURRENT_TIMESTAMP 作为其默认值 |
| `->virtualAs($expression)`       | 创建一个虚拟生成列（MySQL）                            |
| `->generatedAs($expression)`     | 使用指定的序列选项创建标识列（PostgreSQL）             |
| `->always()`                     | 定义序列值优先于标识列的输入（PostgreSQL）             |

### 修改列

#### 先决条件

在修改列之前，确保将 `doctrine/dbal` 依赖项添加到 `composer.json` 文件中。Doctrine DBAL 库用于确定列的当前状态，并创建对列进行指定调整所需的 SQL 查询：

```bash
composer require doctrine/dbal
```

#### 更新列属性

`change` 方法允许你将一些现有列类型修改为新类型或修改列的属性。例如，你可能希望增加字符串列的大小。要查看实际的 `change` 方法，让我们将 `name` 列的大小从 25 增加到 50：

```php
Schema::table('users', function (Blueprint $table) {
    $table->string('name', 50)->change();
});
```

我们还可以将列修改为可空：

```php
Schema::table('users', function (Blueprint $table) {
    $table->string('name', 50)->nullable()->change();
});
```

{% hint style="danger" %}

只有以下列类型可以『更改』：bigInteger、binary、boolean、date、dateTime、dateTimeTz、decimal、integer、json、longText、mediumText、smallInteger、string、text、time、unsignedBigInteger、unsignedInteger 和 unsignedSmallInteger。

{% endhint %}

#### 重命名列

要重命名列，你可以在模式构建器上使用 `renameColumn` 方法。在重命名列之前，确保添加 `doctrine/dbal` 依赖项到 `composer.json` 文件中：

```php
Schema::table('users', function (Blueprint $table) {
    $table->renameColumn('from', 'to');
});
```

{% hint style="danger" %}

当前不支持重命名表中具有 `enum` 类型列的任何列。

{% endhint %}

### 删除列

若要删除列，使用模式生成器上的 `dropColumn` 方法。在从 SQLite 数据库中删除列之前，你需要添加 `doctrine/dbal` 依赖到你的 `composer.json` 文件中，并在你的终端中运行 `composer update` 命令来安装库：

```php
Schema::table('users', function (Blueprint $table) {
    $table->dropColumn('votes');
});
```

通过向 `dropColumn` 方法传递列名数组，你可以从表中删除多个列：

```php
Schema::table('users', function (Blueprint $table) {
    $table->dropColumn(['votes', 'avatar', 'location']);
});
```

{% hint style="danger" %}

不支持在使​​用 SQLite 数据库时在单个迁移中删除或修改多个列。

{% endhint %}

**可用的命令别名**

| 命令                               | 描述                                       |
| ---------------------------------- | ------------------------------------------ |
| `$table->dropMorphs('morphable');` | 删除 `morphable_id` 和 `morphable_type` 列 |
| `$table->dropRememberToken();`     | 删除 `remember_token` 列                   |
| `$table->dropSoftDeletes();`       | 删除 `deleted_at` 列                       |
| `$table->dropSoftDeletesTz();`     | `dropSoftDeletes()` 方法的别名             |
| `$table->dropTimestamps();`        | 删除 `created_at` 和 `updated_at` 列       |
| `$table->dropTimestampsTz();`      | `dropTimestamps()` 方法的别名              |

## 索引

### 创建索引

模式构建器支持几种类型的索引。首先，让我们看一个例子，它指定列的值应该是惟一的。要创建索引，我们可以将 `unique` 方法链接到列定义上：

```php
$table->string('email')->unique();
```

或者，你可以在定义列之后创建索引。例如：

```php
$table->unique('email');
```

你甚至可以将列数组传递给索引方法来创建复合（或组合）索引：

```php
$table->index(['account_id', 'created_at']);
```

Laravel 将自动生成一个合理的索引名称，但是你可以将第二个参数传递给方法来指定自己的名称：

```php
$table->unique('email', 'unique_email');
```

#### 可用的索引类型

每个索引方法都接受一个可选的第二个参数来指定索引的名称。如果省略，名称将从表和列的名称派生。

| 命令                                    | 描述                            |
| --------------------------------------- | ------------------------------- |
| `$table->primary('id');`                | 添加一个主键                    |
| `$table->primary(['id', 'parent_id']);` | 添加组合键                      |
| `$table->unique('email');`              | 添加一个唯一索引                |
| `$table->index('state');`               | 添加一个普通索引                |
| `$table->spatialIndex('location');`     | 添加一个空间索引（除了 SQLite） |

#### 索引长度 & MySQL / MariaDB

Laravel 默认使用 `utf8mb4` 字符集，包括支持在数据库中存储『表情符号』。如果运行的 MySQL 版本比 5.7.7 版本老，或者 MariaDB 版本比 10.2.2 版本老，那么你可能需要手动配置迁移生成的默认字符串长度，以便 MySQL 为它们创建索引。你可以通过在你的 `AppServiceProvider` 中调用 `Schema::defaultStringLength` 方法来配置它：

```php
use Illuminate\Support\Facades\Schema;

/**
 * 引导任何应用程序服务。
 *
 * @return void
 */
public function boot()
{
    Schema::defaultStringLength(191);
}
```

或者，你可以为你的数据库启用 `innodb_large_prefix` 选项。有关如何正确启用此选项的说明，请参阅数据库的文档。

### 重命名索引

要重命名索引，你可以使用 `renameIndex` 方法。此方法接受当前索引名作为其第一个参数，并将期望的名称作为第二个参数：

```bash
$table->renameIndex('from', 'to')
```

### 删除索引

要删除索引，你必须指定索引的名称。默认情况下，Laravel 会自动为索引分配一个合理的名称。将表名、索引列的名称和索引类型连接起来。下面是一些例子：

| 命令                                                     | 描述                                     |
| -------------------------------------------------------- | ---------------------------------------- |
| `$table->dropPrimary('users_id_primary');`               | 从『users』表中删除一个主键              |
| `$table->dropUnique('users_email_unique');`              | 从『users』表中删除唯一索引              |
| `$table->dropIndex('geo_state_index');`                  | 从『geo』表中删除基本索引                |
| `$table->dropSpatialIndex('geo_location_spatialindex');` | 从『geo』表中删除空间索引（SQLite 除外） |

如果将列数组传递给删除索引的方法，则将根据表名、列和键类型生成常规的索引名：

```php
Schema::table('geo', function (Blueprint $table) {
    $table->dropIndex(['state']); // 删除索引「geo_state_index」
});
```

### 外键约束

Laravel 还提供了创建外键约束的支持，这些外键约束用于在数据库级别强制引用完整性。例如，让我们在 `posts` 表上定义一个 `user_id` 列，该列引用 `users` 表上的 `id` 列：

```php
Schema::table('posts', function (Blueprint $table) {
    $table->unsignedBigInteger('user_id');

    $table->foreign('user_id')->references('id')->on('users');
});
```

你还可以为约束的『on delete』和『on update』属性指定期望的操作：

```php
$table->foreign('user_id')
      ->references('id')->on('users')
      ->onDelete('cascade');
```

要删除外键，你可以使用 `dropForeign` 方法。外键约束使用与索引相同的命名约定。因此，我们将把表名和约束中的列连接起来，然后用『_foreign』作为名称的后缀：

```php
$table->dropForeign('posts_user_id_foreign');
```

或者，你可以传递一个数组值，该值在删除时将自动使用常规约束名：

```php
$table->dropForeign(['user_id']);
```

你可以使用以下方法启用或禁用你的迁移中的外键约束：

```php
Schema::enableForeignKeyConstraints();

Schema::disableForeignKeyConstraints();
```

{% hint style="danger" %}

SQLite 默认情况下禁用外键约束。在使用 SQLite 时，尝试在你的迁移中创建外键之前，请确保在你的数据库配置中 [启用外键支持](https://laravel.com/docs/5.8/database#configuration)。

{% endhint %}
