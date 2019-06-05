# Email 认证

## 简介

许多 Web 应用程序要求用户在使用应用程序之前验证他们的 Email 地址。Laravel 为发送和验证电子邮件验证请求提供了方便的方法，而不是强制你在每个应用程序上重新实现此功能。

### 模型准备

首先，验证你的 `App\User` 模型实现了 `Illuminate\Contracts\Auth\MustVerifyEmail` 契约：

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

## 视图

## 验证电子邮件后

## 事件
