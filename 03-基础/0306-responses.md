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

当你调用没有参数的 `redirect` 助手函数时，一个 `Illuminate\Routing\Redirector` 实例被返回，允许你在 `Redirector` 实例上调用任何方法。例如，要对一个命名的路由生成一个 `RedirectResponse`，你可以使用 `route` 方法：

```php
return redirect()->route('login');
```

如果你的路由有参数，你可以将它们作为第二个参数传递到 `route` 方法上：

```php
// 对于具有以下 URI 的路由：profile/{id}

return redirect()->route('profile', ['id' => 1]);
```

#### 通过 Eloquent 模型填充参数

如果你要重定向到一个具有从 Eloquent 模型填充的『ID』参数的路由，你可以传递模型本身。ID 将被自动提取：

```php
// 对于具有以下 URI 的路由：profile/{id}

return redirect()->route('profile', [$user]);
```

如果你想自定义位于路由参数中的值，你应该覆盖在你的 Eloquent 模型上的 `getRouteKey` 方法：

```php
/**
 * 获取模型的路由键值。
 *
 * @return mixed
 */
public function getRouteKey()
{
    return $this->slug;
}
```

### 重定向到控制器动作

你还可以生成重定向到 [控制器动作](https://laravel.com/docs/5.8/controllers)。要这样做，传递控制器和动作的名称到 `action` 方法。记住，你无需去控制器指定完整的命名空间，因为 Laravel 的 `RouteServiceProvider` 将自动设置基控制器的命名空间：

```php
return redirect()->action('HomeController@index');
```

如果控制器路由需要参数，你可以将它们作为第二个参数传递到 `action` 方法：

```php
return redirect()->action(
    'UserController@profile', ['id' => 1]
);
```

### 重定向到外部域名

有时你可能需要重定向到一个应用程序之外的域名。你可以通过 `away` 方法这样做，该方法创建一个 `RedirectResponse` 而无需任何其它的 URL 编码，验证或者认证：

```php
return redirect()->away('https://www.google.com');
```

### 重定向使用闪存会话数据

重定向到一个新的 URL 并 [闪存数据到会话](https://laravel.com/docs/5.8/session#flash-data) 通常是同时完成的。通常，当你闪存一个成功消息到会话，成功执行一个动作之后这个才能完成。为了方便起见，你可以在单个流式方式链上创建一个 `RedirectResponse` 实例并闪存数据到会话：

```php
Route::post('user/profile', function () {
    // 更新用户的个人资料...

    return redirect('dashboard')->with('status', 'Profile updated!');
});
```

用户重定向之后，你可以从 [会话](https://laravel.com/docs/5.8/session) 中显示闪存消息。例如，使用 [Blade 语法](https://laravel.com/docs/5.8/blade)：

```php
@if (session('status'))
    <div class="alert alert-success">
        {{ session('status') }}
    </div>
@endif
```

## 其它响应类型

`response` 助手函数可以用于生成其它类型的响应实例。当 `response` 助手函数在没有参数的情况下调用时，将返回一个 `Illuminate\Contracts\Routing\ResponseFactory` [合约](https://laravel.com/docs/5.8/contracts) 的实现。此合约为生成响应提供几个有用的方法。

### 视图响应

如果你不但需要控制响应的状态和标头，而且还需要去返回一个 [视图](https://laravel.com/docs/5.8/views) 作为响应的内容。你应该使用 `view` 方法：

```php
return response()
            ->view('hello', $data, 200)
            ->header('Content-Type', $type);
```

当然，如果你不需要去传递一个自定义 HTTP 状态码或自定义标头，你应该使用全局的 `view` 助手函数。

### JSON 响应

`json` 方法将自动设置 `Content-Type` 标头为 `application/json`，同时使用 `json_encode` PHP 函数转换给定的数组为 JSON：

```php
return response()->json([
    'name' => 'Abigail',
    'state' => 'CA'
]);
```

如果你想创建一个 JSONP 响应，你可以将 `json` 方法与 `withCallback` 方法结合使用：

```php
return response()
            ->json(['name' => 'Abigail', 'state' => 'CA'])
            ->withCallback($request->input('callback'));
```

### 文件下载

`download` 方法可以用于生成一个强制用户浏览器在给定路径下载文件的响应。`download` 方法接受一个文件名作为方法的第二个参数，该参数将确定下载文件的用户看到文件名。最后，你可以将 HTTP 标头数组作为方法的第三个参数传递：

```php
return response()->download($pathToFile);

return response()->download($pathToFile, $name, $headers);

return response()->download($pathToFile)->deleteFileAfterSend();
```

{% hint style="info" %}

Symfony HttpFoundation，它管理文件下载，要求下载文件具有 ASCII 文件名。

{% endhint %}

#### 流媒体下载

有时你可能希望将给定的操作的字符串转换为可下载的响应而无需将操作的内容写入磁盘。在这个场景中你可以使用 `streamDownload` 方法。此方法接受一个回调，文件名称和一个选项的标头数组作为它的参数：

```php
return response()->streamDownload(function () {
    echo GitHub::api('repo')
                ->contents()
                ->readme('laravel', 'laravel')['contents'];
}, 'laravel-readme.md');
```

### 文件响应

`file` 方法可以用于直接在用户的浏览器显示一个文件，例如，一个图片或 PDF，而不是启动下载。此方法接受文件的路径作为第一个参数并将标头数组作为第二个参数：

```php
return response()->file($pathToFile);

return response()->file($pathToFile, $headers);
```

## 响应宏

如果你想在你的各种路由和控制器中定义一个你能重复使用的自定义响应，你可以在 `Response` facade 上使用 `macro` 方法。例如，从 [服务提供者的](https://laravel.com/docs/5.8/providers) `boot` 方法：

```php
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use Illuminate\Support\Facades\Response;

class ResponseMacroServiceProvider extends ServiceProvider
{
    /**
     * 注册应用程序的响应宏。
     *
     * @return void
     */
    public function boot()
    {
        Response::macro('caps', function ($value) {
            return Response::make(strtoupper($value));
        });
    }
}
```

宏函数接受一个名称作为其第一个参数，并接受 Closure 作为其第二个参数。当从一个 `ResponseFactory` 实现或 `response` 助手函数调用宏名称时，将执行宏的 Closure：

```php
return response()->caps('foo');
```
