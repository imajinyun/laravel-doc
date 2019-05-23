# 认证

## 简介

{% hint style="info" %}

**想快速入门？** 在一个全新的 Laravel 应用程序中只需运行 `php artisan make:auth` 和 `php artisan migrate`。然后，将浏览器导航到 `http：//your-app.test/register` 或分配给你的应用程序的任何其他 URL。这两个命令将负责整个认证系统的脚手架！

{% endhint %}

Laravel 使实现身份验证变得非常简单。事实上，几乎所有东西都是开箱即用的。认证配置文件位于 `config/auth.php`，其中包含几个文档友好的选项，用于调整认证服务的行为。

Laravel 的认证设施的核心是『守卫』和『提供者』。守卫定义了如何为每个请求对用户进行认证。例如：Laravel 附带一个 `session` 防护，它使用会话存储和 `cookie` 来维护状态。

提供者定义如何从持久存储中检索用户。 Laravel 支持使用 Eloquent 和数据库查询构建器检索用户。但是，你可以根据应用程序的需要自由定义其他提供者。

如果这一切听起来有点混乱，请不要担心！许多应用程序永远不需要修改默认的认证配置。

### 数据库注意事项

默认情况下，Laravel 在你的 `app` 目录中包含一个 `App\User` [Eloquent 模型](https://laravel.com/docs/5.8/eloquent)。该模型可与默认的 Eloquent 认证驱动程序一起使用。如果你的应用程序未使用 Eloquent，则可以使用使用 Laravel 查询构建器的 `database` 认证驱动程序。

为 `App\User` 模型构建数据库模式时，请确保密码列的长度至少为 60 个字符。维护 255 个字符的默认字符串列长度是一个不错的选择。

此外，你应该验证你的 `user`（或相当的）表是否包含 100 个字符的可空字符串 `remember_token` 列。此列将用于为登录应用程序时选择『记住我』选项的用户存储令牌。

## 认证快速入门

Laravel 附带了几个预构建的认证控制器，它们位于 `App\Http\Controllers\Auth` 命名空间中。`RegisterController` 处理新用户注册，`LoginController` 处理认证，`ForgotPasswordController` 处理用于重置密码的电子邮件链接，`ResetPasswordController` 包含重置密码的逻辑。这些控制器中的每一个都使用特性来包括它们必要的方法。对于许多应用程序，你根本不需要修改这些控制器。

### 路由

Laravel 提供了一种快速方法，可以使用一个简单的命令来支持你进行认证所需的所有路由和视图：

```bash
php artisan make:auth
```

此命令应当在新应用程序上使用，并将安装布局视图，注册和登录视图以及所有认证端点的路由。还将生成一个 `HomeController` 来处理登录后对应用程序仪表盘的请求。

{% hint style="info" %}

如果你的应用程序不需要注册，我可以通过移除新创建的 `RegisterController` 并修改你的路由定义 `Auth::routes(['register' => false]);` 来禁用它。

{% endhint %}

### 视图

如前所述，`php artisan make:auth` 命令将创建认证需要的所有视图并将它们放置在 `resources/views/auth` 目录下。

`make:auth` 命令还将为应用程序创建一个包含基布局的 `resources/views/layouts` 目录。所有这些视图都使用 Bootstrap CSS 框架，但是你可以随意自定义它们。

### 身份认证

现在你已为所包含的认证控制器设置了路由和视图，你可以为应用程序注册和验证新用户！你可以在浏览器中访问你的应用程序，因为认证控制器已经包含逻辑（通过其特性）来验证现有用户并将新用户存储在数据库中。

#### 路径自定义

当一个用户成功通过身份认证后，他们将被重定向到 `/home` URI。你可以通过在 `LoginController`，`RegisterController`，`ResetPasswordController` 和 `VerificationController` 上定义 `redirectTo` 属性来自定义验证后重定向位置：

```php
protected $redirectTo = '/';
```

接下来，你应该修改 `RedirectIfAuthenticated` 中间件的 `handle` 方法，为了在重定向用户时使用新的 URI。

如果重定向路径需要自定义生成逻辑，可以定义 `redirectTo` 方法，而不是 `redirectTo` 属性：

```php
protected function redirectTo()
{
    return '/path';
}
```

{% hint style="info" %}

`redirectTo` 方法将优先于 `redirectTo` 属性。

{% endhint %}

#### 用户名自定义

默认情况下，`Laravel` 使用 `email` 字段进行身份认证。如果你想自定义它，你可以在 `LoginController` 上定义一个 `username` 方法：

```php
public function username()
{
    return 'username';
}
```

#### 守卫自定义

你还可以自定义用于验证和注册用户的『守卫』。首先，在 `LoginController`、`RegisterController` 和 `ResetPasswordController` 上定义一个守卫方法。方法应该返回一个守卫实例：

```php
use Illuminate\Support\Facades\Auth;

protected function guard()
{
    return Auth::guard('guard-name');
}
```

#### 验证 / 存储自定义

当一个新用户向应用程序注册时要修改所需的表单字段，或自定义如何将新用户存储到数据库中，可以修改 `RegisterController` 类。该类负责验证和创建应用程序的新用户。

`RegisterController` 类的 `validator` 方法为应用程序的新用户包含验证规则。你根据你的需要随意修改这个方法。

`RegisterController` 类的 `create` 方法使用 [Eloquent ORM](https://laravel.com/docs/5.8/eloquent) 负责在你的数据库中创建新的 `App\User` 记录。你根据你的数据库的需要随意修改这个方法。

### 检索认证的用户

你可以通过 `Auth` facade 访问认证的用户：

```php
use Illuminate\Support\Facades\Auth;

// 获取当前认证的用户...
$user = Auth::user();

// 获取当前认证的用户 ID...
$id = Auth::id();
```

或者，一旦用户通过身份认证，您可以通过一个 `Illuminate\Http\Request` 实例访问经过认证的用户。记住，类型提示类将自动注入控制器方法：

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class ProfileController extends Controller
{
    /**
     * 更新用户的个人资料。
     *
     * @param  Request  $request
     * @return Response
     */
    public function update(Request $request)
    {
        // $request->user() 返回一个认证用户的实例...
    }
}
```

#### 确定当前用户是否经过认证

要确定用户是否已经登录到你的应用程序，你可以使用 `Auth` facade 上的 `check` 方法，如果用户已经认证过，该方法将返回 `true`：

```php
use Illuminate\Support\Facades\Auth;

if (Auth::check()) {
    // 用户已登录...
}
```

{% hint style="info" %}

尽管可以使用 `check` 方法确定用户是否已通过认证，但在允许用户访问某些路由 / 控制器之前，通常会使用中间件来验证用户是否已通过身份认证。要了解有关此内容的更多信息，请查看有关 [保护路由](https://laravel.com/docs/5.8/authentication#protecting-routes) 的文档。

{% endhint %}

### 保护路由

[路由中间件](https://laravel.com/docs/5.8/middleware) 可用于只允许认证的用户去访问一个给定的路由。Laravel 附带了一个 `auth` 中间件，该中间件在 `Illuminate\Auth\Middleware\Authenticate` 中定义。由于这个中间件已经注册在你的 HTTP 内核中，你需要做的是将该中间件系到路由定义上：

```php
Route::get('profile', function () {
    // 仅认证的用户可以进入...
})->middleware('auth');
```

如果你使用 [控制器](https://laravel.com/docs/5.8/controllers)，你可以从控制器的构造方法中调用 `middleware` 方法而不是直接将它系在路由定义上：

```php
public function __construct()
{
    $this->middleware('auth');
}
```

#### 重定向未经认证的用户

当 `auth` 中间件检测到一个未认证的用户，它将重定向用户到 `login` [命名的路由](https://laravel.com/docs/5.8/routing#named-routes)。你可以在 `app/Http/Middleware/Authenticate.php` 文件中通过更新 `redirectTo` 方法来修改这个行为：

```php
/**
 * 获取用户应当被重定向的路径。
 *
 * @param  \Illuminate\Http\Request  $request
 * @return string
 */
protected function redirectTo($request)
{
    return route('login');
}
```

#### 指定一个守卫

当将 `auth` 中间件系到一个路由上时，还可以指定应该使用哪个守卫来对用户进行身份认证。指定的守卫应该与 `auth.php` 配置文件的 `guards` 数组中的一个键相对应：

```php
public function __construct()
{
    $this->middleware('auth:api');
}
```

### 登录限制

如果你使用 Laravel 内置的 `LoginController` 类，`Illuminate\Foundation\Auth\ThrottlesLogins` 特性已经包含在你的控制器中。默认情况下，如果多次尝试后用户无法提供正确的凭据，则用户将无法登录一分钟。限制对于用户的用户名 / 电子邮件地址及其 IP 地址是唯一的。

## 手动认证用户

注意，你不需要使用 Laravel 中包含的身份认证控制器。如果选择删除这些控制器，则需要直接使用 Laravel 身份认证类来管理用户认证。别担心，这很容易！

我们将通过 `Auth` facade 访问 Laravel 的身份认证服务，因此我们需要确保在类的顶部导入 `Auth` facade。接下来，让我们检查一下 `attempt` 方法：

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;

class LoginController extends Controller
{
    /**
     * 处理身份认证尝试。
     *
     * @param  \Illuminate\Http\Request $request
     *
     * @return Response
     */
    public function authenticate(Request $request)
    {
        $credentials = $request->only('email', 'password');

        if (Auth::attempt($credentials)) {
            // 认证通过...
            return redirect()->intended('dashboard');
        }
    }
}
```

`attempt` 方法接受一个键 / 值对数组作为其第一个参数。数组中的值将用于查找数据库表中的用户。因此，在上面的示例中，将通过 `email` 列的值检索用户。如果找到用户，则将数据库中存储的散列密码与通过数组传递给方法的 `password` 值进行比较。你不应该将指定的密码哈希为 `password` 值，因为框架会在将它与数据库中的哈希密码进行比较之前自动对其进行哈希处理。如果两个散列密码匹配，则将为用户启动经过身份认证的会话。

如果身份认证成功，则 `attempt` 方法将返回 `true`。 否则，将返回 `false`。

## HTTP 基础认证

## 注销

## 社交认证

## 添加自定义防护

## 添加自定义用户提供者

## 事件
