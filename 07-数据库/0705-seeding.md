# 数据库：播种

## 简介

Laravel 包含一种使用播种类为数据库提供测试数据的简单方法。所有播种类都存储在 `database/seeds` 目录中。播种类可以有任何你想要的名称，但可能应遵循一些合理的约定，例如：`UsersTableSeeder` 等。默认情况下，为你定义了 `DatabaseSeeder` 类。在此类中，你可以使用 `call` 方法运行其他播种类，从而允许你控制播种顺序。

## 编写播种器

要生成播种器，请执行 `make:seeder` [Artisan 命令](https://laravel.com/docs/5.8/artisan)。框架生成的所有播种器都将放在 `database/seeds` 目录中：

```bash
php artisan make:seeder UsersTableSeeder
```

默认情况下，播种器类只包含一个方法：`run`。执行 `db:seed` [Artisan 命令](https://laravel.com/docs/5.8/artisan) 时调用此方法。在 `run` 方法中，你可以随意将数据插入到你的数据库。你可以使用 [查询生成器](https://laravel.com/docs/5.8/queries) 手动插入数据或者你也可以使用有 [Eloquent 模型工厂](https://laravel.com/docs/5.8/database-testing#writing-factories)。

{% hint style="info" %}

在数据库播种期间自动禁用 [批量分配保护](https://laravel.com/docs/5.8/eloquent#mass-assignment)。

{% endhint %}

例如，让我们修改默认的 `DatabaseSeeder` 类并将数据库插入语句添加到 `run` 方法中：

```php
<?php

use Illuminate\Support\Str;
use Illuminate\Database\Seeder;
use Illuminate\Support\Facades\DB;

class DatabaseSeeder extends Seeder
{
    /**
     * 运行数据库播种。
     *
     * @return void
     */
    public function run()
    {
        DB::table('users')->insert([
            'name' => Str::random(10),
            'email' => Str::random(10).'@gmail.com',
            'password' => bcrypt('password'),
        ]);
    }
}
```

{% hint style="info" %}

你可以在 `run` 方法的签名中键入你需要的任何依赖项。它们将通过 Laravel [服务容器](https://laravel.com/docs/5.8/container) 自动解析。

{% endhint %}

### 使用模型工厂

当然，手动指定每个模型播种的属性是很麻烦的。相反，你可以使用 [模型工厂](https://laravel.com/docs/5.8/database-testing#writing-factories) 来方便地生成大量数据库记录。首先，查阅 [模型工厂文档](https://laravel.com/docs/5.8/database-testing#writing-factories)，了解如何定义你的工厂。一旦定义了你的工厂，你就可以使用 `factory` 助手函数将记录插入到你的数据库中。

例如，让我们创建 50 个用户，并为每个用户附加一个关系：

```php
/**
 * 运行数据库播种。
 *
 * @return void
 */
public function run()
{
    factory(App\User::class, 50)->create()->each(function ($user) {
        $user->posts()->save(factory(App\Post::class)->make());
    });
}
```

### 调用额外的播种器

在 `DatabaseSeeder` 类中，你可以使用 `call` 方法执行其它的播种类。使用 `call` 方法可以将数据库播种分解为多个文件，这样就不会有单个的播种器类变得非常大。传递你希望运行的播种器类的名称：

```php
/**
 * 运行数据库播种。
 *
 * @return void
 */
public function run()
{
    $this->call([
        UsersTableSeeder::class,
        PostsTableSeeder::class,
        CommentsTableSeeder::class,
    ]);
}
```

## 运行播种器

编写了你的播种器之后，你可能需要使用 `dump-autoload` 命令重新生成 Composer 的自动加载程序：

```bash
composer dump-autoload
```

现在，你可以使用 `db:seed` Artisan 命令来为数据库播种。默认情况下，`db:seed` 命令运行 `DatabaseSeeder` 类，它可以用来调用其他播种类。但是，你可以使用 `——class` 选项来指定单独运行的特定播种器类：

```bash
php artisan db:seed

php artisan db:seed --class=UsersTableSeeder
```

你还可以使用 `migrate:refresh` 命令来播种你的数据库，该命令还将回滚并重新运行你的所有迁移。此命令对于完全重新构建你的数据库非常有用：

```bash
php artisan migrate:refresh --seed
```

**强制播种器在生产环境中运行**

一些播种操作可能会导致你更改或丢失数据。为了防止对你的生产数据库运行播种命令，在执行播种器之前，将提示你进行确认。若要强制播种者在没有提示的情况下运行，请使用 `-—force` 标志：

```bash
php artisan db:seed --force
```
