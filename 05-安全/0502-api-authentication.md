# API 认证

## 简介

默认情况下，Laravel 通过将一个随机令牌分配给应用程序的每个用户以提供简单的 API 认证解决方案。 在 `config/auth.php` 配置文件中，已经定义了 `api` 守卫并使用了 `token` 驱动程序。此驱动程序负责检查传入请求上的 API 令牌，并验证它是否与数据库中分配给用户令牌匹配。

{% hint style="danger" %}

**注意**：虽然 Laravel 附带一个简单的、基于令牌的身份验证守卫，但是我们强烈建议您考虑使用 [Laravel Passport](https://laravel.com/docs/5.8/passport) 来实现提供 API 认证的健壮的生产应用程序。

{% endhint %}

## 配置

在使用 `token` 驱动程序之前，你需要去 [创建一个迁移](https://laravel.com/docs/5.8/migrations)，并添加一个 `api_token` 列到你的 `users` 表中：

一旦创建了迁移，运行 `migrate` Artisan 命令。

### 数据库准备

```php
Schema::table('users', function ($table) {
    $table->string('api_token', 80)->after('password')
                        ->unique()
                        ->nullable()
                        ->default(null);
});
```

## 生成令牌

一旦将 `api` 令牌列添加到 `users` 表中，就可以对你的应用程序注册的每个用户分配随机的 API 令牌。你应该在注册期间为用户创建一个 `User` 模型时分配这些令牌。当使用 `make:auth` Artisan 命令提供的 [认证脚手架](https://laravel.com/docs/5.8/authentication#authentication-quickstart) 时，这可以在 `RegisterController` 的 `create` 方法中完成：

```php
use Illuminate\Support\Str;
use Illuminate\Support\Facades\Hash;

/**
 * 在有效注册后创建一个新的用户实例。
 *
 * @param  array  $data
 * @return \App\User
 */
protected function create(array $data)
{
    return User::create([
        'name' => $data['name'],
        'email' => $data['email'],
        'password' => Hash::make($data['password']),
        'api_token' => Str::random(60),
    ]);
}
```

### 哈希令牌

在上面的示例中，API 令牌以纯文本形式存储在数据库中。如果你想使用 SHA-256 哈希散列你的 API 令牌，你可以将 `api` 守卫配置的 `hash` 选项设置为 `true`。 `api` 守卫在 `config/auth.php` 配置文件中定义：

```php
'api' => [
    'driver' => 'token',
    'provider' => 'users',
    'hash' => true,
],
```

#### 生成哈希令牌

使用散列 API 令牌时，你不应在用户注册期间生成 `API` 令牌。相反，你需要在应用程序中实现自己的 `API` 令牌管理页面。此页面应允许用户初始化和刷新其 `API` 令牌。当用户发出初始化或刷新其令牌的请求时，你应该在数据库中存储令牌的哈希副本，并将令牌的纯文本副本返回到视图 / 前端客户端以进行一次性显示。

例如，为给定用户初始化 / 刷新令牌并将纯文本令牌作为 JSON 响应返回的控制器方法可能如下所示：

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Support\Str;
use Illuminate\Http\Request;

class ApiTokenController extends Controller
{
    /**
     * 更新已认证用户的 API 令牌。
     *
     * @param  \Illuminate\Http\Request  $request
     * @return array
     */
    public function update(Request $request)
    {
        $token = Str::random(60);

        $request->user()->forceFill([
            'api_token' => hash('sha256', $token),
        ])->save();

        return ['token' => $token];
    }
}
```

{% hint style="info" %}

由于上面示例中的 API 令牌具有足够的信息熵，因此创建『彩虹表』来查找散列令牌的原始值是不切实际的。因此，像 `bcrypt` 这样的慢哈希方法是不必要的。

{% endhint %}

## 保护路由

Laravel 包含一个 [认证守卫](https://laravel.com/docs/5.8/authentication#adding-custom-guards) ，它将在请求传入时自动验证 API 令牌。你仅需要在任何需要有效访问令牌的路由上指定 `auth:api` 中间件：

```php
use Illuminate\Http\Request;

Route::middleware('auth:api')->get('/user', function(Request $request) {
    return $request->user();
});
```

## 在请求中传递令牌

有几种方法可以将 API 令牌传递给你的应用程序。我们将在使用 Guzzle HTTP 库演示它们的用法时讨论这些方法。你可以根据应用程序的需要选择这些方法中的任何一种。

### 查询字符串

你的应用程序的 API 使用者可以将其令牌指定为 `api_token` 查询字符串值：

```php
$response = $client->request('GET', '/api/user?api_token='.$token);
```

### 请求负载

你的应用程序的 API 使用者可以在请求的表单参数中包含其 API 令牌作为 `api_token`：

```php
$response = $client->request('POST', '/api/user', [
    'headers' => [
        'Accept' => 'application/json',
    ],
    'form_params' => [
        'api_token' => $token,
    ],
]);
```

### Bearer 令牌

你的应用程序的 API 使用者可以在请求 `Authorization` 头中提供其 API 令牌作为一个 `Bearer` 令牌

```php
$response = $client->request('POST', '/api/user', [
    'headers' => [
        'Authorization' => 'Bearer '.$token,
        'Accept' => 'application/json',
    ],
]);
```
