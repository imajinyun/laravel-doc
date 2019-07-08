# Email 认证

## 简介

许多 Web 应用程序要求用户在使用应用程序之前验证他们的 Email 地址。Laravel 为发送和验证电子邮件验证请求提供了方便的方法，而不是强制你在每个应用程序上重新实现此功能。

### 模型准备

首先，验证你的 `App\User` 模型实现了 `Illuminate\Contracts\Auth\MustVerifyEmail` 合约：

```php
<?php

namespace App;

use Illuminate\Notifications\Notifiable;
use Illuminate\Contracts\Auth\MustVerifyEmail;
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable implements MustVerifyEmail
{
    use Notifiable;

    // ...
}
```

## 数据库注意事项

### 电子邮件验证列

接下来，`user` 表必须包含一个 `email_verified_at` 列以存储电子邮件地址被验证的日期时间。默认情况下，包含在 Laravel 框架中的 `users` 表迁移已经包含了这一列。因此，你需要做的就是运行数据库迁移：

```bash
php artisan migrate
```

## 路由

Laravel 内置 `Auth\VerificationController` 类，该类包含去发送验证邮件的链接和验证邮件地址所需的逻辑。要为此控制器注册所需的路由，去 `Auth::routes` 方法上传递 `verify` 选项：

```php
Auth::routes(['verify' => true]);
```

### 保护路由

[路由中间件](https://laravel.com/docs/5.8/middleware) 仅可以被用于允许验证的用户去访问一个给定的路由。Laravel 附带一个 `verified` 中间件，它被定义在 `Illuminate\Auth\Middleware\EnsureEmailIsVerified`。由于此中间件已经注册在你的应用程序 HTTP 内核，所以你需要做的就是系这个中间件到路由定义上：

```php
Route::get('profile', function () {
    // 仅验证的用户可以进入...
})->middleware('verified');
```

## 视图

当 `make:auth` 执行时，Laravel 将生成所有必须的电子邮件验证视图。此视图位于 `resources/views/auth/verify.blade.php` 文件中。你根据你的应用程序的需要随意自定义此视图。

## 验证电子邮件之后

验证电子邮件地址后，用户将自动重定向到 `/home`。你可以通过在 `VerificationController` 上定义 `redirectTo` 方法或属性来自定义后验证重定向位置：

```php
protected $redirectTo = '/dashboard';
```

## 事件

Laravel 在电子邮件验证过程中调度 [事件](https://laravel.com/docs/5.8/events)。你可以将侦听器系到 `EventServiceProvider` 中的这些事件上：

```php
/**
 * 应用程序的事件监听器映射。
 *
 * @var array
 */
protected $listen = [
    'Illuminate\Auth\Events\Verified' => [
        'App\Listeners\LogVerifiedUser',
    ],
];
```
