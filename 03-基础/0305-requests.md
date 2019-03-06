# 请求

## 访问请求

要通过依赖注入获取当前 HTTP 请求实例，你应该在你的控制器方法上键入类型提示 `Illuminate\Http\Request` 类。即将到来的请求实例将自动通过 [服务容器](https://laravel.com/docs/5.8/container) 注入：

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class UserController extends Controller
{
    /**
     * 存储一个新用户。
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        $name = $request->input('name');

        //
    }
}
```

**依赖注入 & 路由参数**

如果你的控制器方法也期望从一个路由参数中输入，你应该在其它依赖之后列出路由参数。例如，如果你的路由像这样定义：

```php
Route::put('user/{id}', 'UserController@update');
```

你仍然可以通过定义你的控制器方法键入类型提示 `Illuminate\Http\Request` 并访问你的路由参数 `id`，如下所示：

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class UserController extends Controller
{
    /**
     * 更新指定的用户。
     *
     * @param  Request  $request
     * @param  string  $id
     * @return Response
     */
    public function update(Request $request, $id)
    {
        //
    }
}
```

**通过路由闭包访问路由**

你还可以在一个路由 Closure 上键入类型提示 `Illuminate\Http\Request` 类。服务容器将它执行时自动注入即将到来的请求到 Closure：

```php
use Illuminate\Http\Request;

Route::get('/', function (Request $request) {
    //
});
```

### 请求路径 & 方法

`Illuminate\Http\Request` 实例提供了各种方法来检查应用程序的 HTTP 请求检查，并继承了 `Symfony\Component\HttpFoundation\Request` 类。下面我们将讨论一些重要的方法。

#### 检索请求的路径

`path` 方法返回请求的路径信息。因此，如果即将到来的请求目标是 `http://domain.com/foo/bar`，`path` 文件将返回 `foo/bar`：

```php
$uri = $request->path();
```

`is` 方法允许你去验证即将到来的请求路径是否匹配一个给定的模式。在使用此方法时你可以使用 `*` 字符作为一个通配符：

```php
if ($request->is('admin/*')) {
    //
}
```

#### 检索请求的 URL

你可以使用 `url` 或 `fullUrl` 方法去检索即将到来的请求的完整 URL。`url` 方法将返回没有查询字符串的 URL，而 `fullUrl` 方法包含查询字符串：

```php
// 没有查询字符串...
$url = $request->url();

// 使用查询字符串...
$url = $request->fullUrl();
```

#### 检索请求方法

`method` 方法将返回请求的 HTTP 动词。你可以使用 `isMethod` 就去去验证 HTTP 动词是否匹配一个给定的字符串：

```php
$method = $request->method();

if ($request->isMethod('post')) {
    //
}
```

### PSR-7 请求

[PSR-7 标准](https://www.php-fig.org/psr/psr-7/) 指定 HTTP 消息的接口，包括请求和响应。如果你想获取一个 PSR-7 请求实例而不是 Laravel 请求的实例，你首先需要安装一些库。Laravel 使用 *Symfony Http Message Bridge* 组件将典型的 Laravel 请求和响应转换成 PSR-7 兼容的实现：

```php
composer require symfony/psr-http-message-bridge
composer require zendframework/zend-diactoros
```

一旦你安装这些库，你可以通过在你的路由 Closure 或控制器方法上键入类型提示的请求接口获取一个 PSR-7 请求：

```php
use Psr\Http\Message\ServerRequestInterface;

Route::get('/', function (ServerRequestInterface $request) {
    //
});
```

{% hint style="info" %}

如果你从一个路由或控制器返回一个 PSR-7 响应实例，它将自动转换回 Laravel 响应实例并由框架显示。

{% endhint %}

## 输入修整和标准化

默认情况下，Laravel 在应用程序的全局中间件堆栈中包含了 `TrimStrings` 和 `ConvertEmptyStringToNull` 中间件。这些中间件由 `App\Http\Kernel` 类列在堆栈中。这些中间件将自动修整请求上的所有传入字符串字段，也将任何空字符串字段转换为 `null`。这允许你不必在你的路由和控制器中担心这些规范化问题。

如果你想禁用此行为，你可以从你的应用程序中间件堆栈中移除这两个中间件，即从 `App\Http\Kernel` 类的 `$middleware` 属性中移除。

## 检索输入

* **检索所有输入数据**

你还可以使用 `all` 方法检索所有输入数据到一个 `array` 中：

```php
$input = $request->all();
```

* **检索一个输入值**

使用一些简单的方法，你可以从你的 `Illuminate\Http\Request` 实例访问所有用户的输入而不担心请求使用了哪个 HTTP 动词。无论 HTTP 动词如何，`input` 方法都可用于检索用户的输入：

```php
$name = $request->input('name');
```

你可以传递一个默认的值作为第二个参数到 `input` 方法。如果在请求上请求的值不存在此值将被返回：

```php
$name = $request->input('name', 'Sally');
```

当请求的表单包含数组输入时，使用『点』表示法去访问数组：

```php
$name = $request->input('products.0.name');

$names = $request->input('products.*.name');
```

为了检索所有的输入值作为一个关联数组，你可以调用 `input` 方法而无需任何参数：

```php
$input = $request->input();
```

* **从查询字符串中检索输入**

虽然 `input` 方法从整个请求负载（包括查询字符串）中检索值，`query` 方法将仅从查询字符串中检索值：

```php
$name = $request->query('name');
```

如果请求的查询字符串值数据不存在，将返回此方法的第二个参数：

```php
$name = $request->query('name', 'Helen');
```

为了检索所有的查询字符串值作为一个关联数组，你可以调用 `query` 方法而无需任何参数：

```php
$query = $request->query();
```

* **通过动态属性检索输入**

你也可以在 `Illuminate\Http\Request` 实例上使用动态属性访问用户的输入。例如，如果你的应用程序的表单包含一个 `name` 字段，你可以像这样访问这个字段的值：

```php
$name = $request->name;
```

当使用动态属性时，Laravel 将首先在请求的负载中查找参数的值。如果它不存在，Laravel 将在路由参数中搜索该字段。

* **检索 JSON 输入值**

当发送 JSON 请求到你的应用程序，只要请求头的 `Content-Type` 正确设置为 `application/json`，你就可以通过 `input` 方法访问 JSON 数据。你甚至可以使用『点』语法去挖掘 JSON 数组：

```php
$name = $request->input('user.name');
```

* **检索输入数据的部分**

如果你需要检索输入数据的子集，可以使用 `only` 和 `except` 方法。这两种方法都接受单个 `array` 或者一个动态参数列表：

```php
$input = $request->only(['username', 'password']);

$input = $request->only('username', 'password');

$input = $request->except(['credit_card']);

$input = $request->except('credit_card');
```

{% hint style="info" %}

`only` 方法返回请求上的所有键 / 值对；然而，它不能返回在请求上不存在的键 / 值对。

{% endhint %}

* **确定输入值是否存在**

你应该使用 `has` 方法去确定请求上的一个值是否存在。如果请求上存在该值，`has` 方法返回 `true`：

```php
if ($request->has('name')) {
    //
}
```

当给定的一个数组时，`has` 方法将确定所有指定的值是否存在：

```php
if ($request->has(['name', 'email'])) {
    //
}
```

如果你想确定请求上的一个值存在并且不为空，你可以使用 `filled` 方法：

```php
if ($request->filled('name')) {
    //
}
```

### 旧输入

Laravel 允许你从一个请求到下一个请求期间保持输入。这个功能对检测验证错误后重新填充表单是非常有用的。然而，如果你使用 Laravel 包含的 [验证功能](https://laravel.com/docs/5.8/validation)，你不太可能需要手动使用这些方法，因为一些 Laravel 的内置验证工具将自动调用它们。

#### 闪存输入到会话

在 `Illuminate\Http\Request` 类上的 `flash` 方法将闪存当前输入到 [会话](https://laravel.com/docs/5.8/session)，以便于用户的下一个请求到应用程序时可用：

```php
$request->flash();
```

你也可以使用 `flashOnly` 和 `flashExcept` 方法去闪存一个请求数据的子集到会话。这些方法将密码等敏感信息保留在会话之外：

```php
$request->flashOnly(['username', 'email']);

$request->flashExcept('password');
```

#### 闪存输入到重定向

由于你经常把输入闪存到会话并重定向到之前的页面，你可以使用 `withInput` 方法轻地将输入闪存链接到一个重定向上：

```php
return redirect('form')->withInput();

return redirect('form')->withInput(
    $request->except('password')
);
```

#### 检索旧输入

要从之前的请求中检索闪存输入，使用 `Request` 实例上的 `old` 方法。`old` 方法将从 [会话](https://laravel.com/docs/5.8/session) 中提取之前闪存的输入数据：

```php
$username = $request->old('username');
```

Laravel 也提供一个全局的 `old` 助手函数。如果你要在一个 [Blade 模板](https://laravel.com/docs/5.8/blade) 中显示旧输入，使用 `old` 助手函数会更方便。如果给定的字段不存在旧输入时，则返回 `null`：

```php
<input type="text" name="username" value="{{ old('username') }}">
```

### Cookies

#### 从请求检索 Cookies

通过 Laravel 框架创建的所有 cookie 都要加密并使用一个认证的代码签名，这意味着如果客户端更改了它们将视为无效。要从请求中检索一个 cookie 的值，在 `Illuminate\Http\Request` 实例上使用 `cookie` 方法：

```php
$value = $request->cookie('name');
```

或者，你可以使用 `Cookie` facade 去访问 cookie 的值：

```php
$value = Cookie::get('name');
```

### 将 Cookie 附加到响应

你可以使用 `cookie` 方法附加一个 cookie 到传出的 `Illuminate\Http\Response` 实例。你应当传递 cookie 应视为有效的名称，值和分钟数到这个方法：

```php
return response('Hello World')->cookie(
    'name', 'value', $minutes
);
```

`cookie` 方法也接受一些较少的不频繁使用的参数。通常，这些参数与 PHP 的原生 [setcookie](https://secure.php.net/manual/en/function.setcookie.php) 函数给定的参数具有相同的目的和意义：

```php
return response('Hello World')->cookie(
    'name', 'value', $minutes, $path, $domain, $secure, $httpOnly
);
```

或者，你可以使用 `Cookie` facade 去『排队』cookie 以附加到应用程序传出的响应上。`queue` 方法接受一个 `Cookie` 实例或创建 `Cookie` 实例所需的参数。这些 cookie 将在发送到浏览器之前附加到传出的响应上：

```php
Cookie::queue(Cookie::make('name', 'value', $minutes));

Cookie::queue('name', 'value', $minutes);
```

### 生成 Cookie 实例

如果你想生成一个 `Symfony\Component\HttpFoundation\Cookie` 实例，该实例可以在以后提供给响应实例，你可以使用全局的 `cookie` 助手函数。此 cookie 不会被发送回客户端，除非它被附加到一个响应实例上：

```php
$cookie = cookie('name', 'value', $minutes);

return response('Hello World')->cookie($cookie);
```

## 文件

### 检索上传文件

你可以使用 `file` 方法或者使用动态属性从一个 `Illuminate\Http\Request` 实例上访问上传的文件。`file` 方法返回一个 `Illuminate\Http\UploadedFile` 类示例，此类继承了 PHP `SplFileInfo` 类并提供了与文件交互的各种方法：

```php
$file = $request->file('photo');

$file = $request->photo;
```

你可以使用 `hasFile` 方法确定请求中是否存在一个文件：

```php
if ($request->hasFile('photo')) {
    //
}
```

#### 验证成功上传

除了检查文件是否存在之外，你可以通过 `isValid` 方法验证上传的文件有没有问题：

```php
if ($request->file('photo')->isValid()) {
    //
}
```

#### 文件路径 & 扩展

`UploadedFile` 类也包含了访问文件的完整路径和它的扩展的方法。`extension` 方法将尝试基于文件的内容去猜测文件的扩展。此扩展可以可能与客户端提供的扩展不同：

```php
$path = $request->photo->path();

$extension = $request->photo->extension();
```

#### 其它文件方法

`UploadedFile` 实例上有许多其它可用的方法。有关这些方法的更多信息查看 [此类的 API 文档](https://api.symfony.com/3.0/Symfony/Component/HttpFoundation/File/UploadedFile.html)。

### 存储上传文件

要存储一个上传的文件，通常你将使用一个你配置的 [文件系统](https://laravel.com/docs/5.8/filesystem)。`UploadedFile` 类有一个 `store` 方法，它将移动上传文件到你的某个磁盘上，磁盘将位于你的本地文件系统或者甚至是一个像 Amazon S3 一样的云存储。

`store` 方法接受相对于文件系统的配置的根目录存储的文件路径。此路径应该不包含文件名称，由于一个唯一的 ID 将自动生成充当文件名称。

`store` 方法还接受一个可选的第二个参数，此参数将用于存储文件的磁盘名称。该方法将返回相对于磁盘根目录的文件路径：

```php
$path = $request->photo->store('images');

$path = $request->photo->store('images', 's3');
```

如果你不想自动生成文件名称，你可以使用 `storeAs` 方法，此方法接受路径，文件名称和磁盘名称作为其参数：

```php
$path = $request->photo->storeAs('images', 'filename.jpg');

$path = $request->photo->storeAs('images', 'filename.jpg', 's3');
```

## 配置信任代理

当你的应用程序运行在终止 TLS / SSL 证书的负载均衡器后面时，你可能会注意到你的应用程序有时不会生成 HTTPS 链接。这通常是因为你的应用程序正在从 80 端口上的负载均衡程序器转发流量并且不知道它应该生成安全链接。

为了解决这个问题，你可以使用 `App\Http\Middleware\TrustProxies` 中间件，此中间件包含在你的 Laravel 应用程序中，它允许你快速自定义应用程序应该信任的负载均衡或代理。你信任的代理应该在此中间件的 `$proxies` 属性上作为一个数组列出来。除了配置信任代理之外，你可以配置应该被信任的代理 `$headers`：

```php
<?php

namespace App\Http\Middleware;

use Illuminate\Http\Request;
use Fideloper\Proxy\TrustProxies as Middleware;

class TrustProxies extends Middleware
{
    /**
     * 此应用程序的受信任代理。
     *
     * @var array
     */
    protected $proxies = [
        '192.168.1.1',
        '192.168.1.2',
    ];

    /**
     * 用于检测代理的头文件。
     *
     * @var string
     */
    protected $headers = Request::HEADER_X_FORWARDED_ALL;
}
```

### 信任所有代理

```php
/**
 * 此应用程序的受信的代理。
 *
 * @var array
 */
protected $proxies = '*';
```
