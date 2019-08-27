# 数据库测试

## 简介

Laravel 提供了各种有用的工具，使测试你的数据库驱动的应用程序更加容易。首先，你可以使用 `assertDatabaseHas` 助手来断言数据库中存在匹配给定标准集的数据。例如，如果你想验证 `users` 表中是否有一条电子邮件值为 `sally@example.com` 记录，你可以执行以下操作：

```php
public function testDatabase()
{
    // 调用应用程序...

    $this->assertDatabaseHas('users', [
        'email' => 'sally@example.com'
    ]);
}
```

***

你也可以使用 `assertDatabaseMissing` 助手去断言数据库中不存在数据。

`assertDatabaseHas` 方法和其他类似的助手是为了方便。你可以自由地使用 PHPUnit 的任何内置断言方法来补充你的测试。

## 生成工厂

要创建一个工厂，使用 `make:factory` [Artisan 命令](https://laravel.com/docs/5.8/artisan)：

```bash
php artisan make:factory PostFactory
```

***

新的工厂将位于你的 `database/factories` 目录。

`--model` 选项可以用于指示通过工厂创建的方法名称。此选项将使用给定模型预填充生成的工厂文件：

```bash
php artisan make:factory PostFactory --model=Post
```

***

## 每次测试后重置数据库

在每次测试之后重置数据库通常很有用，以便于之前测试的数据就不会干扰后续的测试。`RefreshDatabase` 特性采用最优的方法迁移你的测试数据库，这取决于你是使用内存数据库还是传统数据库。在你的测试类上使用特性，一切都会为你处理：

```php
<?php

namespace Tests\Feature;

use Tests\TestCase;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Illuminate\Foundation\Testing\WithoutMiddleware;

class ExampleTest extends TestCase
{
    use RefreshDatabase;

    /**
     * A basic functional test example.
     *
     * @return void
     */
    public function testBasicExample()
    {
        $response = $this->get('/');

        // ...
    }
}
```

***

## 编写工厂

测试时，你可能需要在执行你的测试之前将几条记录插入数据库。在创建此测试数据时，不是手动指定每列的值，而是 Laravel 允许你使用模型工厂为每个 [Eloquent 模型](https://laravel.com/docs/5.8/eloquent) 定义一组默认属性。要开始，查看你的应用程序中的 `database/factories/UserFactory.php` 文件。 开箱即用，此文件包含一个工厂定义：

```php
use Illuminate\Support\Str;
use Faker\Generator as Faker;

$factory->define(App\User::class, function (Faker $faker) {
    return [
        'name' => $faker->name,
        'email' => $faker->unique()->safeEmail,
        'email_verified_at' => now(),
        'password' => '$2y$10$TKh8H1.PfQx37YgCzwiKb.KjNyWgaHb9cbcoQgdIVFlYg7B77UdFm', // secret
        'remember_token' => Str::random(10),
    ];
});
```

***

在充当工厂定义的闭包中，你可以返回模型上所有属性的默认测试值。闭包将接收 [Faker](https://github.com/fzaninotto/Faker) PHP 库的一个实例，该实例允许你方便地生成各种随机数据进行测试。

你还可以为每个模型创建其他工厂文件，以便更好地进行组织。例如，你可以在 `database/factories` 目录中创建 `UserFactory.php` 和 `CommentFactory.php` 文件。`factories` 目录中的所有文件都将由 Laravel 自动加载。

{% hint style="info" %}

你可以通过在 `config/app.php` 配置文件中添加 `faker_locale` 选项来设置 Faker 区域设置。

{% endhint %}

### 工厂状态

状态允许你定义可以应用于任何组合中的模型工厂的不连续修改。例如，你的 `User` 模型可能具有修改其默认属性值之一的 `delinquent` 状态。你可以使用 `state` 方法定义你的状态转换。对于简单状态，可以传递属性数组修改：

```php
$factory->state(App\User::class, 'delinquent', [
    'account_status' => 'delinquent',
]);
```

***

如果你的状态需要计算或 `$faker` 实例，你可以使用闭包计算状态的属性修改：

```php
$factory->state(App\User::class, 'address', function ($faker) {
    return [
        'address' => $faker->address,
    ];
});
```

***

### 工厂回调

工厂回调是使用 `afterMaking` 和 `aftercreate` 方法注册的，允许你在创建或创建模型之后执行其他任务。例如，你可以使用回调将其他模型与创建的模型关联起来：

```php
$factory->afterMaking(App\User::class, function ($user, $faker) {
    // ...
});

$factory->afterCreating(App\User::class, function ($user, $faker) {
    $user->accounts()->save(factory(App\Account::class)->make());
});
```

***

你还可以为 [工厂状态](https://laravel.com/docs/5.8/database-testing#factory-states) 定义回调：

```php
$factory->afterMakingState(App\User::class, 'delinquent', function ($user, $faker) {
    // ...
});

$factory->afterCreatingState(App\User::class, 'delinquent', function ($user, $faker) {
    // ...
});
```

***

## 使用工厂

### 创建模型

## 使用播种

## 可用的断言
