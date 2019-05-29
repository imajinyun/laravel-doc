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

[HTTP 基础认证](https://en.wikipedia.org/wiki/Basic_access_authentication) 提供一个快速的方式去认证你的应用程序的用户，而无需设计一个专门的『登录』页面。首先，把 `auth.basic` [中间件](https://laravel.com/docs/5.8/middleware) 系到你的路由上。`auth.basic` 中间件包含在 Laravel 框架中，所以你不需要定义它：

```php
Route::get('profile', function () {
    // 仅认证可能进入的用户...
})->middleware('auth.basic');
```

一旦中间件被附加到路由上，当你在浏览器中访问路由时，将自动提示你输入凭据。默认情况下，是 `auth.basic` 中间件将使用用户记录上的 `email` 列作为『用户名』。

**关于 FastCGI 的说明**

如果你使用 PHP FastCGI，HTTP 基础认证可能无法立即正确工作。应该将以下行添加到 `.htaccess` 文件中：

```bash
RewriteCond %{HTTP:Authorization} ^(.+)$
RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]
```

### 无状态 HTTP 基础认证

你也可以使用 HTTP 基础认证，而无需在会话中设置用户标识符 cookie，这对 API 认证特别有用。为此，请定义一个调用 `onceBasic` 方法的 [中间件](https://laravel.com/docs/5.8/middleware)。 如果 `onceBasic` 方法没有返回任何响应，则该请求可以进一步传递到应用程序中：

```php
<?php

namespace App\Http\Middleware;

use Illuminate\Support\Facades\Auth;

class AuthenticateOnceWithBasicAuth
{
    /**
     * 处理传入的请求。
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @return mixed
     */
    public function handle($request, $next)
    {
        return Auth::onceBasic() ?: $next($request);
    }

}
```

接下来，[注册路由中间件](https://laravel.com/docs/5.8/middleware#registering-middleware) 并将其系到一个路由上：

```php
Route::get('api/user', function () {
    // 只有经过认证的用户才能进入...
})->middleware('auth.basic.once');
```

## 注销

要手动将用户从应用程序中注销，可以使用 `Auth` facade 上的 `logout` 方法。这将清除用户会话中的认证信息：

```php
use Illuminate\Support\Facades\Auth;

Auth::logout();
```

### 使其他设备上的会话无效

Laravel 还提供了一种机制，用于在其他设备上激活和『注销』用户会话，而不会在他们当前设备上的会话无效。在开始之前，你应确保在你的 `app/Http/Kernel.php` 类的 `web` 中间件组中存在 `Illuminate\Session\Middleware\AuthenticateSession` 中间件并取消注释：

```php
'web' => [
    // ...
    \Illuminate\Session\Middleware\AuthenticateSession::class,
    // ...
],
```

然后，你可以使用 `Auth` facade 上的 `logoutOtherDevices` 方法。此方法要求用户提供其当前密码，应用程序应该通过输入表单接受该密码：

```php
use Illuminate\Support\Facades\Auth;

Auth::logoutOtherDevices($password);
```

{% hint style="danger" %}

当调用 `logoutOtherDevices` 方法时，用户的其他会话将完全失效，这意味着它们将从之前通过认证的所有守卫中『注销』。

{% endhint %}

## [社交认证](https://github.com/laravel/socialite)

## 添加自定义防护

你可以使用 `Auth` 外观上的 `extend` 方法定义自己的认证守卫。你应该将此调用放在一个 [服务提供者](https://laravel.com/docs/5.8/providers) 中进行 `extend`。由于 Laravel 已经附带了 `AuthServiceProvider`，我们可以将代码放在该提供程序中：

```php
<?php

namespace App\Providers;

use App\Services\Auth\JwtGuard;
use Illuminate\Support\Facades\Auth;
use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;

class AuthServiceProvider extends ServiceProvider
{
    /**
     * 注册任何应用程序认证 / 授权服务。
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Auth::extend('jwt', function ($app, $name, array $config) {
            // 返回一个 Illuminate\Contracts\Auth\Guard 实例...

            return new JwtGuard(Auth::createUserProvider($config['provider']));
        });
    }
}
```

正如你在上面的示例中所看到的，传递给 `extend` 方法的回调应该返回一个 `Illuminate\Contracts\Auth\Guard` 的实现。这个接口包含一些方法，你需要实现这些方法来定义自定义守卫。一旦定义了自定义守卫，就可以在 `auth.php` 配置文件的 `guards` 配置中使用该守卫：

```php
'guards' => [
    'api' => [
        'driver' => 'jwt',
        'provider' => 'users',
    ],
],
```

### 关闭请求守卫

实现基于 HTTP 请求的自定义认证系统的最简单方法是使用 `Auth::viaRequest` 方法。此方法允许你使用单个 `Closure` 快速定义认证过程。

首先，在 `AuthServiceProvider` 的 `boot` 方法中调用 `Auth::viaRequest` 方法。`viaRequest` 方法接受认证驱动程序名称作为它的第一个参数。此名称可以是描述自定义守卫的任何字符串。传递给方法的第二个参数应该是一个闭包，它接收传入的 HTTP 请求并返回一个用户实例，如果认证失败，则返回 `null`：

```php
use App\User;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;

/**
 * 注册任何应用程序认证 / 授权服务。
 *
 * @return void
 */
public function boot()
{
    $this->registerPolicies();

    Auth::viaRequest('custom-token', function ($request) {
        return User::where('token', $request->token)->first();
    });
}
```

一旦定义了自定义认证驱动程序，就可以将其用作 `auth.php` 配置文件的 `guards` 配置的一个驱动程序：

```php
'guards' => [
    'api' => [
        'driver' => 'custom-token',
    ],
],
```

## 添加自定义用户提供者

如果不使用传统的关系数据库存储用户，则需要使用自己的认证用户提供程序扩展 Laravel。我们将使用 `Auth` facade 上的 `provider` 方法来定义一个自定义用户提供者：

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Auth;
use App\Extensions\RiakUserProvider;
use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;

class AuthServiceProvider extends ServiceProvider
{
    /**
     * 注册任何应用程序认证 / 授权服务。
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Auth::provider('riak', function ($app, array $config) {
            // 返回一个 Illuminate\Contracts\Auth\UserProvider 实例...

            return new RiakUserProvider($app->make('riak.connection'));
        });
    }
}
```

使用 `provider` 方法注册了提供者之后，你可以在 `auth.php` 配置文件中切换到新的用户提供者。首先，定义一个使用新驱动程序的 `provider`：

```php
'providers' => [
    'users' => [
        'driver' => 'riak',
    ],
],
```

最后，可以在 `guards` 配置中使用此提供程序：

```php
'guards' => [
    'web' => [
        'driver' => 'session',
        'provider' => 'users',
    ],
],
```

### 用户提供者契约

`Illuminate\Contracts\Auth\UserProvider` 实现仅负责从持久存储系统（如：MySQL，Riak 等）中获取一个 `Illuminate\Contracts\Auth\Authenticatable` 实现。这两个接口允许 Laravel 认证机制继续起作用，而不管用户数据是如何存储的，或者使用什么类型的类来表示它。

让我们来看一下 `Illuminate\Contracts\Auth\UserProvider` 契约：

```php
<?php

namespace Illuminate\Contracts\Auth;

interface UserProvider {

    public function retrieveById($identifier);
    public function retrieveByToken($identifier, $token);
    public function updateRememberToken(Authenticatable $user, $token);
    public function retrieveByCredentials(array $credentials);
    public function validateCredentials(Authenticatable $user, array $credentials);

}
```

`retrieveById` 函数通常接收表示用户的密钥，例如来自 MySQL 数据库的自增 ID。应该检索并返回与该 ID 匹配的 `Authenticatable` 实现。

`retrieveByToken` 方法通过用户唯一的 `$identifier` 和存储在 `remember_token` 字段上的『记住我』`$token` 检索用户。和之前的方法一样，应当返回 `Authenticatable` 的实现。

`updateRememberToken` 方法用一个新的 `$token` 更新 `$user` 字段 `remember_token`。在成功的『记住我』登录尝试或当用户退出时分配一个新的令牌。

`retrieveByCredentials` 方法接收当尝试登录应用程序时传递给 `Auth::attempt` 方法的凭据数组。然后，该方法应该『查询』匹配这些凭证的用户的底层持久存储。通常，该方法将运行一个 `$credentials['username']` 上带有『条件』的查询。然后，该方法应该返回 `Authenticatable` 的实现。**此方法不应尝试执行任何密码验证或认证。**

`validateCredentials` 方法应该将给定的 `$user` 与 `$credentials` 进行比较以对用户进行认证。例如，该方法可能应该使用 `Hash::check` 来比较 `$user->getAuthPassword()` 和 `$credentials['password']` 的值。此方法应返回 `true` 或 `false` 以表明密码是否有效。

### 认证契约

现在，我们已经研究了 `UserProvider` 上的每个方法，让我们来看看 `Authenticatable` 契约。请记住，提供者应该从 `retrieveById`，`retrieveByToken` 和 `retrieveByCredentials` 方法返回此接口的实现：

```php
<?php

namespace Illuminate\Contracts\Auth;

interface Authenticatable {

    public function getAuthIdentifierName();
    public function getAuthIdentifier();
    public function getAuthPassword();
    public function getRememberToken();
    public function setRememberToken($value);
    public function getRememberTokenName();

}
```

这个接口很简单。`getAuthIdentifierName` 方法应该返回用户的『主键』字段的名称，`getAuthIdentifier` 方法应该返回用户的『主键』。同样，在 MySQL 后端，这将是自增的主键。`getAuthPassword` 应该返回用户的散列密码。这个接口允许认证系统与任何用户类一起工作，不管你使用的是什么 ORM 或存储抽象层。默认情况下，Laravel 在 `app` 目录中包含实现该接口的一个 `User` 类，所以你可以参考这个类来实现一个示例。

## 事件
