# 哈希

## 简介

Laravel `Hash` [facade](https://laravel.com/docs/5.8/facades) 提供了用于存储用户密码的安全 Bcrypt 和 Argon2 哈希。如果你正在使用 Laravel 应用程序中包含的内置 `LoginController` 和 `RegisterController` 类，默认情况下它们将使用 Bcrypt 进行注册和认证。

{% hint style="info" %}

Bcrypt 是哈希密码的一个很好的选择，因为它的『工作因子』是可调的，这意味着生成哈希的时间可以随着硬件功率的增加而增加。

{% endhint %}

## 配置

应用程序的默认哈希驱动程序在 `config/hashing.php` 配置文件中配置。目前有三种支持的驱动程序：[Bcrypt](https://en.wikipedia.org/wiki/Bcrypt) 和 [Argon2](https://en.wikipedia.org/wiki/Argon2)（Argon2i 和 Argon2id 变体）。

{% hint style="danger" %}

Argon2i 驱动程序需要 PHP 7.2.0 或更高版本，而 Argon2id 驱动程序需要 PHP 7.3.0 或更高版本。

{% endhint %}

## 基本用法

你可以通过调用 `Hash` facade 上的 `make` 方法来哈希一个密码：

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Hash;
use App\Http\Controllers\Controller;

class UpdatePasswordController extends Controller
{
    /**
     * 更新用户密码。
     *
     * @param  Request  $request
     * @return Response
     */
    public function update(Request $request)
    {
        // 验证新密码的长度...

        $request->user()->fill([
            'password' => Hash::make($request->newPassword)
        ])->save();
    }
}
```

### 调整 Bcrypt 工作因子

如果你正在使用 Bcrypt 算法，`make` 方法允许你使用 `round` 选项管理算法的工作因子；但是，默认值对于大多数应用程序都是可以接受的：

```php
$hashed = Hash::make('password', [
    'rounds' => 12
]);
```

### 调整 Argon2 工作因子

如果你正在使用 Argon2 算法，`make` 方法允许你使用 `memory`、`time` 和 `threads` 选项管理算法的工作因子；但是，默认值对于大多数应用程序都是可以接受的：

```php
$hashed = Hash::make('password', [
    'memory' => 1024,
    'time' => 2,
    'threads' => 2,
]);
```

{% hint style="info" %}

有关这些选项的更多信息，请查看 [官方 PHP 文档](https://secure.php.net/manual/en/function.password-hash.php)。

{% endhint %}

### 对照哈希验证密码

`check` 方法允许你验证给定的纯文本字符串是否对应于给定的哈希。但是，如果你正在使用 [Laravel 中包含](https://laravel.com/docs/5.8/authentication) 的 `LoginController`，你可能不需要直接使用它，因为这个控制器会自动调用这个方法：

```php
if (Hash::check('plain-text', $hashedPassword)) {
    // 密码匹配...
}
```

### 检查密码是否需要重新哈希

`needsRehash` 函数允许你确定通过 hasher 使用的工作因子是否在密码哈希之后发生了更改：

```php
if (Hash::needsRehash($hashed)) {
    $hashed = Hash::make('plain-text');
}
```
