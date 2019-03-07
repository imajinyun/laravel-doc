# 响应

## 创建响应

### 字符串 & 数组

所有路由和控制器应该返回一个响应以发送回用户的浏览器。Laravel 提供几个不同的方式去返回响应。最基本的响应是从一个路由或者控制器中返回一个字符串。框架将自动把字符串转换成一个完整的 HTTP 响应：

```php
Route::get('/', function () {
    return 'Hello World';
});
```

除了从你的路由和控制器返回字符串之外，你还可以返回数组。框架将自动把数组转换为一个 JSON 响应：

```php
Route::get('/', function () {
    return [1, 2, 3];
});
```

{% hint style="info" %}

你是否知道你还可以从你的路由或控制器返回 [Eloquent 集合](https://laravel.com/docs/5.8/eloquent-collections)？它们将自动转换为 JSON。试试看！

{% endhint %}

### 响应对象

通过，你不只从你的路由动作返回简单的字符串或数组。相反，你将返回完整的 `Illuminate\Http\Response` 实例或 [视图](https://laravel.com/docs/5.8/views)。

返回完整的 `Response` 实例允许你去自定义响应的 HTTP 状态码和标头。一个 `Response` 实例继承自 `Symfony\Component\HttpFoundation\Response` 类，它提供了各种方法构建 HTTP 响应的方法：

```php
Route::get('home', function () {
    return response('Hello World', 200)
                  ->header('Content-Type', 'text/plain');
});
```

### 附加头到响应

记住，大多数响应方法是可链式调用的，允许流式构建响应实例。例如，你可以在发送回用户之前使用 `header` 方法去添加一系列标头到响应上：

```php
return response($content)
            ->header('Content-Type', $type)
            ->header('X-Header-One', 'Header Value')
            ->header('X-Header-Two', 'Header Value');
```

或者，你可以使用 `withHeaders` 方法指定要添加到响应的一个标头数组：

```php
return response($content)
            ->withHeaders([
                'Content-Type' => $type,
                'X-Header-One' => 'Header Value',
                'X-Header-Two' => 'Header Value',
            ]);
```

### 附加 Cookie 到响应

响应实例上的 `cookie` 方法允许你轻松附加 cookie 到响应上。例如，你可以使用 `cookie` 方法生成一个 cookie 并像这样流式附加它到响应实例：

```php
return response($content)
                ->header('Content-Type', $type)
                ->cookie('name', 'value', $minutes);
```

`cookie` 方法也接受一些较少频繁使用的参数。通常，这些参数与 PHP 的原生 [setcookie](https://secure.php.net/manual/en/function.setcookie.php) 函数给定的参数具有相同的目的和意义：

```php
->cookie($name, $value, $minutes, $path, $domain, $secure, $httpOnly)
```

或者，你可以使用 `Cookie` facade 去『排队』cookie 以附加到应用程序传出的响应上。`queue` 方法接受一个 `Cookie` 实例或创建一个 `Cookie` 实例的所需的参数。这些 cookie 将在发送到浏览器之前附加到传出的响应上：

```php
Cookie::queue(Cookie::make('name', 'value', $minutes));

Cookie::queue('name', 'value', $minutes);
```

### Cookie & 加密

默认情况下，Laravel 生成的所有 cookie 要经过加密和签名，以便于通过客户端不能修改或读取它们。如果你想禁用经由应用程序生成的 cookie 子集加密，你可以使用 `App\Http\Middleware\EncryptCookies` 中间件的 `$except` 属性，它位于 `app/Http/Middleware` 目录中：

```php
/**
 * 不应该加密的 cookie 名称。
 *
 * @var array
 */
protected $except = [
    'cookie_name',
];
```

## 重定向

重定向响应是 `Illuminate\Http\RedirectResponse` 类的实例，并包含将用户重定向到另一个 URL 所需的正确标头。有几种方式去生成一个 `RedirectResponse` 实例。最简单的方法是使用全局的 `redirect` 助手函数：

```php
Route::get('dashboard', function () {
    return redirect('home/dashboard');
});
```

有时你希望去重定向用到到他们之前的位置，比如当一个提交的表单是无效的时候。你可以使用全局的 `back` 助手函数来这样做。由于此功能利用 [会话](https://laravel.com/docs/5.8/session)，确保路由调用的 `back` 函数使用 `web` 中间件组或已应用所有会话中间件：

```php
Route::post('user/profile', function () {
    // 验证请求...

    return back()->withInput();
});
```

### 重定向到命名路由

### 重定向到控制器动作

### 重定向到扩展域名

### 重定向使用闪存会话数据

## 其它响应类型

## 响应宏
