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

## 运行播种器
