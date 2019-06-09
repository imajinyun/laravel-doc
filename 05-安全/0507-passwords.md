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

一旦定义了路由和视图以重置用户的密码，你就可以在你的浏览器中访问 `/password/reset` 路由。框架中包含的 `ForgotPasswordController` 已经包含发送密码重置链接电子邮件的逻辑，而 `ResetPasswordController` 包含重置用户密码的逻辑。

重置密码后，用户将自动登录到应用程序并重定向到 `/home`。你可以通过在 `ResetPasswordController` 上定义一个 `redirectTo` 属性来自定义密码重置后重定向的位置：

```php
protected $redirectTo = '/dashboard';
```

{% hint style="danger" %}

默认情况下，密码重置令牌在一小时后过期。你可以通过 `config/auth.php` 文件中的密码重置 `expire` 选项更改此设置。

{% endhint %}

## 自定义

### 认证保护定制

在你的 `auth.php` 配置文件中，你可以配置多个『守卫』，这些『守卫』可用于为多个用户表定义认证行为。你可以通过覆盖控制器上的 `guard` 方法自定义包含的 `ResetPasswordController` 来使用你选择的守卫。这个方法应该返回一个守卫实例：

```php
use Illuminate\Support\Facades\Auth;

/**
 * 获取密码重置期间使用的守卫。
 *
 * @return \Illuminate\Contracts\Auth\StatefulGuard
 */
protected function guard()
{
    return Auth::guard('guard-name');
}
```

### 密码代理定制

在你的 `auth.php` 配置文件中，可以配置多个密码『代理』，这些代理可用于重置多个用户表上的密码。你可以通过覆盖 `broker` 方法自定义包含的 `ForgotPasswordController` 和 `ResetPasswordController` 来使用你选择的代理：

```php
use Illuminate\Support\Facades\Password;

/**
 * 获取密码重置期间使用的代理。
 *
 * @return PasswordBroker
 */
public function broker()
{
    return Password::broker('name');
}
```

### 重置电子邮件定制

你可以轻松地修改用于向用户发送密码重置链接的通知类。首先，覆盖 `User` 模型上的 `sendPasswordResetNotification` 方法。在此方法中，可以使用选择的任何通知类发送通知。密码重置 `$token` 是该方法接收的第一个参数：

```php
/**
 * 发送密码重置通知。
 *
 * @param  string  $token
 * @return void
 */
public function sendPasswordResetNotification($token)
{
    $this->notify(new ResetPasswordNotification($token));
}
```
