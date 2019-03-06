# 中间件

* [简介](#jian-jie)
* [定义中间件](#ding-yi-zhong-jian-jian)
* [注册中间件](#zhu-ce-zhong-jian-jian)
  * [全局中间件](#quan-ju-zhong-jian-jian)
  * [分配中间件到路由](#fen-pei-zhong-jian-jian-dao-lu-you)
  * [中间件组](#zhong-jian-jian-zu)
  * [排序中间件组](#pai-xu-zhong-jian-jian)
* [中间件参数](#zhong-jian-jian-can-shu)
* [可终止中间件](#ke-zhong-zhi-zhong-jian-jian)

## 简介

中间件提供了一种方便的机制来过滤进入应用程序的 HTTP 请求。例如，Laravel 包含一个验证应用程序用户身份认证的中间件。如果用户没有认证，中间件将重定向用户到登录界面。但是，如果用户认证过，中间件将允许请求进一步进入应用程序。

除了认证之外，还可以编写额外的中间件来执行各种任务。一个 CORS 中间件可能负责为离开你的应用程序的的所有响应头添加正确的标头。日志记录中间件可能记录应用程序的所有即将到来的请求。

有几个中间件包含在 Laravel 框架内，包括用户身份认证和 CSRF 保护中间件。所有的这些中间件位于 `app/Http/Middleware` 目录中。

## 定义中间件

使用 `make:middleware` Artisan 命令创建一个新的中间件：

```php
php artisan make:middleware CheckAge
```

此命令将在你的 `app/Http/Middleware` 目录中放置一个新的 `CheckAge` 类。在此中间件中，如果提供的 `age` 大于 200，我们将仅允许访问路由。否则，我们将用户重定向回到 `home` URI：

```php
<?php

namespace App\Http\Middleware;

use Closure;

class CheckAge
{
    /**
     * 处理即将到来的请求。
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @return mixed
     */
    public function handle($request, Closure $next)
    {
        if ($request->age <= 200) {
            return redirect('home');
        }

        return $next($request);
    }
}
```

如你所见，如果你给定 `age` 是小于或等于 200，此中间件将返回一个 HTTP 重定向到客户端；否则，请求将进一步传递到应用程序中。要将请求更深入地传递到应用程序中（允许中间件去『传递』），使用 `$request` 调用 `$next` 回调。

最好将中间件想像成 HTTP 请求必须在他们到达应用程序之前穿过的一系列『层』，通过。每个层可能检查请求并甚至完全拒绝。

{% hint style="info" %}

所有中间件都通过 [服务容器] 解析，因此你可以在中间件的构建方法中键入任何你需要的依赖项的类型提示。

{% endhint %}

### 前置 & 后置中间件

一个中间件运行在一个请求的之前还是之后取决于中间件本身。例如，如下的中间件将在应用程序处理请求 **之前** 执行一些任务：

```php
<?php

namespace App\Http\Middleware;

use Closure;

class BeforeMiddleware
{
    public function handle($request, Closure $next)
    {
        // 执行动作

        return $next($request);
    }
}
```

然而，此中间件将在应用程序处理请求 **之后** 执行其任务：

```php
<?php

namespace App\Http\Middleware;

use Closure;

class AfterMiddleware
{
    public function handle($request, Closure $next)
    {
        $response = $next($request);

        // 执行动作

        return $response;
    }
}
```

## 注册中间件

### 全局中间件

如果你想让一个中间件运行在应用程序的每个 HTTP 请求期间，在你的 `app/Http/Kernel.php` 类的 `$middleware` 属性列出中间件类。

### 分配中间件到路由

如果你想分配中间件到指定的路由，你应该首先在 `app/Http/Kernel.php` 文件中给中间件分配下个键名。默认情况下，此类的 `$routeMiddleware` 属性包含 Laravel 包括的整个中间件。要添加你自己的，添加它到此列表中并分配给它一个你选择的键名：

```php
// 在 App\Http\Kernel 类...

protected $routeMiddleware = [
    'auth' => \App\Http\Middleware\Authenticate::class,
    'auth.basic' => \Illuminate\Auth\Middleware\AuthenticateWithBasicAuth::class,
    'bindings' => \Illuminate\Routing\Middleware\SubstituteBindings::class,
    'cache.headers' => \Illuminate\Http\Middleware\SetCacheHeaders::class,
    'can' => \Illuminate\Auth\Middleware\Authorize::class,
    'guest' => \App\Http\Middleware\RedirectIfAuthenticated::class,
    'signed' => \Illuminate\Routing\Middleware\ValidateSignature::class,
    'throttle' => \Illuminate\Routing\Middleware\ThrottleRequests::class,
    'verified' => \Illuminate\Auth\Middleware\EnsureEmailIsVerified::class,
];
```

一旦中间件已经定义在 HTTP 内核中，你可以使用 `middleware` 方法去分配中间件到一个路由上：

```php
Route::get('admin/profile', function () {
    //
})->middleware('auth');
```

你还可以分配多个中间件到路由上：

```php
Route::get('/', function () {
    //
})->middleware('first', 'second');
```

当分配中间件时，你还可以传递完全限定的类名：

```php
use App\Http\Middleware\CheckAge;

Route::get('admin/profile', function () {
    //
})->middleware(CheckAge::class);
```

### 中间件组

有时你可能想组合几个中间件到一个键名下使它们更容易地分配到路由上。你可以使用 HTTP 内核的 `$middlewareGroups` 属性来这样做。

Laravel 附带的 `web` 和 `api` 中间件组开箱即用，其中包含你可能想应用到你的 web UI 和 API 路由的公共中间件：

```php
/**
 * 应用程序的路由中间件组。
 *
 * @var array
 */
protected $middlewareGroups = [
    'web' => [
        \App\Http\Middleware\EncryptCookies::class,
        \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
        \Illuminate\Session\Middleware\StartSession::class,
        \Illuminate\View\Middleware\ShareErrorsFromSession::class,
        \App\Http\Middleware\VerifyCsrfToken::class,
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
    ],

    'api' => [
        'throttle:60,1',
        'auth:api',
    ],
];
```

可以使用与单个中间件相同的语法将中间件分配到路由和控制器动作上。同样，中间件组使得一次分配许多中间件到一个路由更加方便：

```php
Route::get('/', function () {
    //
})->middleware('web');

Route::group(['middleware' => ['web']], function () {
    //
});
```

{% hint style="info" %}

开箱即用，`web` 中间件组通过 `RouteServiceProvider` 类将自动应用到 `routes/web.php` 文件。

{% endhint %}

### 排序中间件

少有的情况下，你可能需要中间件以一个特定的顺序去执行，但它们分配到路由时无法控制它们的顺序。在这种情况下，你可以在 `app/Http/Kernel.php` 文件中使用 `$middlewarePriority` 属性指定你的中间件优先级：

```php
/**
 * 中间件列表的优先级排序。
 *
 * 这会强制非全局中间件始终按给定的顺序排列。
 *
 * @var array
 */
protected $middlewarePriority = [
    \Illuminate\Session\Middleware\StartSession::class,
    \Illuminate\View\Middleware\ShareErrorsFromSession::class,
    \App\Http\Middleware\Authenticate::class,
    \Illuminate\Session\Middleware\AuthenticateSession::class,
    \Illuminate\Routing\Middleware\SubstituteBindings::class,
    \Illuminate\Auth\Middleware\Authorize::class,
];
```

## 中间件参数

中间件还可心接受额外的参数。例如，如果你的应用程序在执行给定的动作之前需要验证认证的用户是否具有一个给定的『角色』，你可以创建一个 `CheckRole` 中间件，它接受一个角色名称作为一个额外参数。

在 `$next` 参数之后，额外的中间件参数将传递到中间件：

```php
<?php

namespace App\Http\Middleware;

use Closure;

class CheckRole
{
    /**
     * 处理即将到来的请求。
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @param  string  $role
     * @return mixed
     */
    public function handle($request, Closure $next, $role)
    {
        if (! $request->user()->hasRole($role)) {
            // Redirect...
        }

        return $next($request);
    }

}
```

当定义路由通过用 `:` 分隔中间件名称和参数时，可以指定中间件参数。多个参数应当通过逗号来分隔：

```php
Route::put('post/{id}', function ($id) {
    //
})->middleware('role:editor');
```

## 可终止中间件

有时一个中间件可能需要在 HTTP 响应准备好之后要做某些工作。例如， Laravel 包含的『会话』中间件在响应完全准备好之后将会话数据写入存储。如果你在你的中间件上定义一个 `terminate` 方法，则在响应准备发送到浏览器之后将自动调用它。

```php
<?php

namespace Illuminate\Session\Middleware;

use Closure;

class StartSession
{
    public function handle($request, Closure $next)
    {
        return $next($request);
    }

    public function terminate($request, $response)
    {
        // 存储 session 数据...
    }
}
```

`terminate` 方法应该同时接受请求和响应。一旦你定义了一个可终止中间件，你应该添加它到路由列表或者在 `app/Http/Kernel.php` 文件全局中间件中。

当在你的中间件上调用 `terminate` 方法，Laravel 将从 [服务容器](https://laravel.com/docs/5.8/container) 中解析一个中间件新实例。如果要在调用 `handle` 和 `terminate` 方法时你喜欢使用相同的中间件实例，使用容器的 `singleton` 方法向容器注册中间件。
