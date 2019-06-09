# 重置密码

## 简介

{% hint style="info" %}

**想快速入门？**仅需要在新的 Laravel 应用程序中运行 `php artisan make:auth`，然后在浏览器中导航到 `http://your-app.test/register` 或分配给你的应用程序的任何其他 URL。这个单一命令将负责整个认证系统的脚手架，包括重置密码！

{% endhint %}

大多数 Web 应用程序都为用户提供了一种重置密码的方法。与强制你在每个应用程序上重新实现此功能不同，Laravel 提供了发送密码提醒和执行密码重置的方便方法。

{% hint style="danger" %}

使用 Laravel 的密码重置功能之前，你的用户必须使用 `Illuminate\Notifications\Notifiable` 特性。

{% endhint %}

## 数据库注意事项

首先，验证你的 `App\User` 模型实现 `Illuminate\Contracts\Auth\CanResetPassword` 契约。框架中包含的 `App\User` 模型已经实现了此接口，并使用 `Illuminate\Auth\Passwords\CanResetPassword` 特性来包含实现该接口所需的方法。

### 生成重置令牌表迁移

接下来，必须创建一个表来存储密码重置令牌。此表的迁移包含在 Laravel 中，开箱即用，并存在于 `database/migrations` 目录中。因此，你需要做的就是运行数据库迁移：

```bash
php artisan migrate
```

## 路由

Laravel 包含 `Auth\ForgotPasswordController` 和 `Auth\ResetPasswordController` 类，它们包含电子邮件密码重置链接和重置用户密码所需的逻辑。可以使用 `make:auth` Artisan 命令生成执行密码重置所需的所有路由：

```bash
php artisan make:auth
```

## 视图

再次，当执行 `make:auth` 命令时，Laravel 将生成密码重置的所有必需视图。这些视图位于 `resources/views/auth/passwords` 中。你可以根据应用程序的需要随意定制它们。

## 重置密码之后

## 自定义
