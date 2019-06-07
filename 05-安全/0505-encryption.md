# 数据加密

## 简介

Laravel 的加密机使用 OpenSSL 提供 AES-256 和 AES-128 加密。强烈建议你使用 Laravel 内置的加密设施，而不要尝试使用你自己的『如法炮制的』加密算法。Laravel 的所有加密值都使用消息认证代码（MAC）签名，因此一旦加密，就不能修改它们的基础值。

## 配置

在使用 Laravel 的加密器之前，必须在 `config/app.php` 配置文件中设置一个 `key` 选项。你应当使用 `php artisan key:generate` 命令来生成此密钥，因为此 Artisan 命令将使用 PHP 的安全随机字节生成器来构建你的密钥。如果未正确设置此值，则 Laravel 加密的所有值都将不安全。

## 使用加密

### 加密一个值

你可以使用 `encrypt` 助手加密一个值。所有加密值都使用 OpenSSL 和 `AES-256-CBC` 加密。此外，所有加密值都使用消息认证代码（MAC）签名，以检测对加密字符串的任何修改：

```php
<?php

namespace App\Http\Controllers;

use App\User;
use Illuminate\Http\Request;
use App\Http\Controllers\Controller;

class UserController extends Controller
{
    /**
     * 为用户存储私密消息。
     *
     * @param  Request  $request
     * @param  int  $id
     * @return Response
     */
    public function storeSecret(Request $request, $id)
    {
        $user = User::findOrFail($id);

        $user->fill([
            'secret' => encrypt($request->secret)
        ])->save();
    }
}
```

### 无序列化加密

加密期间，加密值通过 `serialize` 传递，允许对对象和数组进行加密。因此，非 PHP 客户端接收加密值的将需要 `unserialize` 数据。如果希望加密和解密值而不进行序列化，可以使用 `Crypt` facade 的 `encryptString` 和 `decryptString` 方法：

```php
use Illuminate\Support\Facades\Crypt;

$encrypted = Crypt::encryptString('Hello world.');

$decrypted = Crypt::decryptString($encrypted);
```

### 解密一个值

你可以使用 `decrypt` 助手解密值。如果该值不能正确解密，例如当 MAC 无效时，将抛出一个 `Illuminate\Contracts\Encryption\DecryptException` 异常：

```php
use Illuminate\Contracts\Encryption\DecryptException;

try {
    $decrypted = decrypt($encryptedValue);
} catch (DecryptException $e) {
    //
}
```
