# 授权

## 简介

除了提供开箱即用的 [认证](https://laravel.com/docs/5.8/authentication) 服务之外，Laravel 还提供了一种简单的方法来授权用户对给定资源的操作。与认证一样，Laravel 的授权方法也很简单，有两种主要的授权操作方法：gates 和策略。

想想 Gates 和策略，比如路由和控制器。Gates 提供了一种简单的、基于闭包的授权方法，而策略（如控制器）则根据特定的模型或资源对其逻辑进行分组。我们将首先探究 Gates，然后探究策略。

在构建应用程序时，你不需要在专用 Gates 和专用策略之间进行选择。大多数应用程序很可能包含 Gates 和策略的混合，这非常好！Gates 最适用于与任何模型或资源无关的动作，例如：查看管理员仪表板。相反，当你希望授权特定模型或资源的动作时，应该使用策略。

## Gates

### 编写 Gates

Gates 是决定用户是否被授权执行给定动作的闭包，通常使用的 `Gate` facade 的在 `App\Providers\AuthServiceProvider` 类中定义。Gates 总是接收用户实例作为其第一个参数，并且可以选择接收其他参数，比如相关的 Eloquent 模型：

```php
/**
 * 注册任何认证 / 授权服务。
 *
 * @return void
 */
public function boot()
{
    $this->registerPolicies();

    Gate::define('update-post', function ($user, $post) {
        return $user->id == $post->user_id;
    });
}
```

Gates 还可以使用一个 `Class@method` 样式的回调字符串定义，像控制器：

```php
/**
 * 注册任何认证 / 授权服务。
 *
 * @return void
 */
public function boot()
{
    $this->registerPolicies();

    Gate::define('update-post', 'App\Policies\PostPolicy@update');
}
```

### 授权动作

要授权使用 Gates 的动作，你应该使用 `allows` 或 `denies` 方法。注意，不需要将当前经过认证的用户传递给这些方法。Laravel 自动将用户传递到 Gate 的 Closure：

```php
if (Gate::allows('update-post', $post)) {
    // 当前用户能可以新文章...
}

if (Gate::denies('update-post', $post)) {
    // 当前用户不能更新文章...
}
```

如果你想确定某个特定用户是否被授权执行某个动作，可以在 `Gate` facade 上使用 `forUser` 方法：

```php
if (Gate::forUser($user)->allows('update-post', $post)) {
    // 当前用户能可以新文章...
}

if (Gate::forUser($user)->denies('update-post', $post)) {
    // 当前用户不能更新文章...
}
```

### Gate 拦截检查

有时，你可能希望将所有能力授予特定的用户。你可以使用 `before` 方法来定义一个在所有其他授权检查之前运行的回调：

```php
Gate::before(function ($user, $ability) {
    if ($user->isSuperAdmin()) {
        return true;
    }
});
```

如果 `before` 回调返回非空结果，则该结果将被视为检查的结果。

你可以使用 `after` 方法去定义一个在所有其它授权检查之后执行的回调：

```php
Gate::after(function ($user, $ability, $result, $arguments) {
    if ($user->isSuperAdmin()) {
        return true;
    }
});
```

类似于 `before` 检查，如果 `after` 回调所有一个非空结果，则该结果将被视为检查的结果。

## 创建策略

### 生成策略

策略是围绕特定模型或资源组织授权逻辑的类。例如，如果你的应用程序是一个博客，你可能有一个 `Post 模型和相应的 `PostPolicy` 来授权用户动作（比如创建或更新帖子）。

你可以使用 `make:policy` [Artisan 命令](https://laravel.com/docs/5.8/artisan) 生成一个策略。生成的策略位于 `app/Policies` 目录中。如果这个目录在你的应用程序中不存在，Laravel 将为你创建它：

```bash
php artisan make:policy PostPolicy
```

`make:policy` 命令将生成一个空策略类。如果你希望生成一个类，该类中已经包含基本的『CRUD』策略方法，你可以在执行命令时指定一个 `--model` 参数：

```bash
php artisan make:policy PostPolicy --model=Post
```

{% hint style="info" %}

所有的策略通过 Laravel [服务容器](https://laravel.com/docs/5.8/container) 去解析，允许你在策略构造中键入类型提示任何需要的依赖，它们将被自动注入。

{% endhint %}

### 注册策略

一旦策略存在，就需要注册它。新 Laravel 应用程序中包含的 `AuthServiceProvider` 包含一个 `policy` 属性，它将你的 Eloquent 模型映射到它们相应的策略上。注册策略将指示 Laravel 在授权针对给定模型的动作时利用哪个策略：

```php
<?php

namespace App\Providers;

use App\Post;
use App\Policies\PostPolicy;
use Illuminate\Support\Facades\Gate;
use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;

class AuthServiceProvider extends ServiceProvider
{
    /**
     * 应用程序的策略映射。
     *
     * @var array
     */
    protected $policies = [
        Post::class => PostPolicy::class,
    ];

    /**
     * 注册任何应用程序认证 / 授权服务。
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        //
    }
}
```

#### 策略自动发现

只要模型和策略遵循标准的 Laravel 命名约定，Laravel 就可以自动发现策略，而不是手动注册模型策略。具体而言，策略必须位于包含模型的目录下的 `Policies` 目录中。因此，例如，模型可以放在 `app` 目录中，而策略可以放在 `app/Policies` 目录中。此外，策略名称必须与模型名称匹配，并具有策略后缀。因此，`User` 模型将对应于 `UserPolicy` 类。

## 编写策略

### 策略模型

### 无模型的方法

### Guest 用户

### 策略过滤

## 使用策略授权动作

### 通过用户模型

### 通过中间件

### 通过控制器助手

### 通过 Blade 模版
