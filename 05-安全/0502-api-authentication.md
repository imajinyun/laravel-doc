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

## 保护路由

## 在请求中传递令牌
