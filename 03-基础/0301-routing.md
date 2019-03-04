# 路由

## 基本路由

最基本的 Laravel 路由接受一个 URL 和一个 `Closure`，提供一个非常简单且定义路由的表达性强的方法：

```php
Route::get('foo', function () {
    return 'Hello World';
});
```

**默认路由文件**

所有 Laravel 路由都定义在路由文件中，这些路由位于 `routes` 目录中。这个文件由框架自动加载。`routes/web.php` 文件定义 web 界面的路由。这个文件中定义的路由被分配给 web 中间件组，它提供了会话状态和 CSRF 保护等功能。routes/api.php 定义了无状态的路由并被分配给 api 中间件组。

对于大多数的应用程序，你首先将在你的 `routes/web.php` 文件中定义路由。定义在 `routes/web.php` 文件中的路由可以通过在你的浏览器中输入定义的路由 URL 去访问。例如，你可以通过在浏览器中导航到 `http://your-app.test/user` 地址来访问如下的路由：

```php
Route::get('/user', 'UserController@index');
```

定义在 `routes/api.php` 文件中的路由通过 `RouteServiceProvider` 嵌套在路由组中。在该组中，`/api` URI 前缀是自动应用的，因此你无需手动应用它到文件中的每个路由。你可以通过修改 `RouteServiceProvider` 类来修改前缀和其它路由组选项。

**可用路由方法**

路由器允许你去注册响应任何 HTTP 动词的路由：

```php
Route::get($uri, $callback);
Route::post($uri, $callback);
Route::put($uri, $callback);
Route::patch($uri, $callback);
Route::delete($uri, $callback);
Route::options($uri, $callback);
```

有时你可能需要注册一个响应到多个 HTTP 动词的路由。你可以使用 `match` 方法这样做。或者，你甚至可以使用 `any` 方法注册一个响应到所有 HTTP 动词的路由：

```php
Route::match(['get', 'post'], '/', function () {
    //
});

Route::any('foo', function () {
    //
});
```

**CSRF 保护**

在 `web` 路由文件中定义的指向 `POST`，`PUT` 或 `DELETE` 路由的任何 HTML 表单应当包含一个 CSRF 令牌字段。否则，请求将被拒绝。有关更多的 CSRF 保护你可以阅读 [CSRF 文档](https://laravel.com/docs/5.8/csrf)：

```php
<form method="POST" action="/profile">
    @csrf
    ...
</form>
```

### 路由重定向

如果你要定义一个重定向到另一个 URI 的路由，你可以使用 `Route::redirect` 方法。此方法提供了方便的快捷方式以便于你无需为执行一个简单重定向去定义完事路由或控制器：

```php
Route::redirect('/here', '/there');
```

默认情况下，`Route::redirect` 返回一个 `302` 状态码。你可以使用第三个参数选项自定义状态码：

```php
Route::redirect('/here', '/there', 301);
```

你可以使用 `Route::permanentRedirect` 方法去返回一个 `302` 状态码：

```php
Route::permanentRedirect('/here', 'there');
```

### 视图路由

如果你的路由只需要返回一个视图，你可以使用 `Route::view` 方法。像 `redirect` 方法，此方法提供一个简单的快捷方式以便于你无需定义一个完整的路由或控制器。`view` 方法接受一个 URI 作为它的第一个参数并将视图名称作为它的第二个参数。另外，你可以提供一个数组数据作为可选的第三个参数传递到视图：

```php
Route::view('/welcome', 'welcome');

Route::view('/welcome', 'welcome', ['name' => 'Taylor']);
```

## 路由参数

### 必要参数

有时你将需要去捕获你的路由中的 URI 段。例如，你可能需要从一个 URL 中捕获一个用户的 ID。你可以通过定义路由参数来这样做：

```php
Route::get('user/{id}', function ($id) {
    return 'User '.$id;
});
```

你可以定义与许多路由参数一样多的必要的路由：

```php
Route::get('posts/{post}/comments/{comment}', function ($postId, $commentId) {
    //
});
```

路由参数始终封闭在 `{}` 括号内，并且应该由字母字符组成且不能包含一个 `-` 字段。使用一个下划线（`_`）而不是 `-` 字符。路由参数被注入到路由回调 / 控制器是基于它们的顺序——与回调 / 控制器参数的名称无关紧要。

### 可选参数

偶尔你可能需要指定一个路由参数，但是要让该路由参数存在是可选的。你可以通过在参数名称后放置一个 `?` 标记来这样做。确保对给定的路由的相应变量赋予一个默认值：

```php
Route::get('user/{name?}', function ($name = null) {
    return $name;
});

Route::get('user/{name?}', function ($name = 'John') {
    return $name;
});
```

### 正则表达式约束

你可以在路由实例上使用 `where` 方法约束路由参数的格式。`where` 方法接受参数的名称和参数应该如何约束的一个正则表达式定义：

```php
Route::get('user/{name}', function ($name) {
    //
})->where('name', '[A-Za-z]+');

Route::get('user/{id}', function ($id) {
    //
})->where('id', '[0-9]+');

Route::get('user/{id}/{name}', function ($id, $name) {
    //
})->where(['id' => '[0-9]+', 'name' => '[a-z]+']);
```

#### 全局约束

如果你希望一个路由参数始终通过一个给定的正则表达式约束，则可以使用 `pattern` 方法。你应该在你的 `RouteServiceProvider` 类的 `boot` 方法中定义这些模式：

```php
/**
 * 定义你的路由模型绑定，模式过滤等等...
 *
 * @return void
 */
public function boot()
{
    Route::pattern('id', '[0-9]+');

    parent::boot();
}
```

一旦模式被定义，它将自动应用到所有使用该参数名称的路由上：

```php
Route::get('user/{id}', function ($id) {
    // 仅在 {id} 是数字时执行...
});
```

#### 编码正斜杠

Laravel 路由组件允许除 `/` 之外的所有字符。你必须使用 `where` 条件正则表达式明确地允许 `/` 成为占位符的一部分：

```php
Route::get('search/{search}', function ($search) {
    return $search;
})->where('search', '.*');
```

{% hint sytle="danger" %}

仅在最后的路由段中支持编码的正斜杠。

{% endhint %}

## 命名路由

命名路允许为特定的路由方便地生成 URL 或重定向。你可以通过链式 `name` 方法在路由的定义上指定一个路由的名称：

```php
Route::get('user/profile', function () {
    //
})->name('profile');
```

你还可以为控制器动作指定路由名称：

```php
Route::get('user/profile', 'UserProfileController@show')->name('profile');
```

### 对命名路由生成 URL

一旦你对一个给定的路由分配一个名称，当通过全局的 `route` 方法生成 URL 或重定向时，你可以使用路由的名称：

```php
// 生成 URL...
$url = route('profile');

// 生成重定向...
return redirect()->route('profile');
```

如果命名的路由定义了参数，你可以将参数作为第二个参数佬到 `route` 函数。给定的参数将自动插入到 URL 中的正确位置：

```php
Route::get('user/{id}/profile', function ($id) {
    //
})->name('profile');

$url = route('profile', ['id' => 1]);
```

### 检索当前路由

如果你想确定当前请求是否已路由到一个给定的命名路由，你可以在一个路由实例上使用 `named` 方法。例如，你可以从一个路由中间件中检查当前路由名称：

```php
/**
 * 处理即将到来的请求。
 *
 * @param  \Illuminate\Http\Request  $request
 * @param  \Closure  $next
 * @return mixed
 */
public function handle($request, Closure $next)
{
    if ($request->route()->named('profile')) {
        //
    }

    return $next($request);
}
```

## 路由组

路由组允许你去共享路由属性，比如：中间件或命名空间，穿过大量的路由而无需在每个路由上去定义这些属性。共享属性被指定在一个数组格式中作为 `Route::group` 方法的第一个参数。

嵌套组尝试智能地将属性与它们的父组『合并』。中间件和 `where` 条件合并的同时附加了名称、命名空间和前缀。在 URI 前缀适当的位置自动添加命名空间分隔符和斜杠。

### 中间件

要分配中间件到组内的所有路由，你可以在定义组之前使用 `middleware` 方法。中间件按它们在数组中列出的顺序执行：

```php
Route::middleware(['first', 'second'])->group(function () {
    Route::get('/', function () {
        // 使用第一 & 第二个中间件
    });

    Route::get('user/profile', function () {
        // 使用第一 & 第二个中间件
    });
});
```

### 命名空间

路由组的另一个常见使用情形时使用 `namespace` 方法将相同的 PHP 命名空间分配给一组控制器：

```php
Route::namespace('Admin')->group(function () {
    // 以 "App\Http\Controllers\Admin" 为命名空间的控制器
});
```

记住，默认情况下，`RouteServiceProvider` 包含你的命名空间组中的路由文件，允许你去注册控制器路由而无需指定完整 `App\Http\Controllers` 命名空间前缀。因此，你仅需要去指定基本的 `App\Http\Controllers` 命名空间之后的命名空间部分即可。

### 子域名路由

路由组还可以用来处理子域名路由。可以像路由 URL 一样为子域名分配路由参数，允许你去捕获子域名的一部分为了在你的路由或控制器中使用。子域名大定义组之前可以通过调用 `domain` 方法去指定：

```php
Route::domain('{account}.myapp.com')->group(function () {
    Route::get('user/{id}', function ($account, $id) {
        //
    });
});
```

### 路由前缀

`prefix` 方法可以用于为组中的每个路由添加一个给定的 URI。例如，你可以给组内的所有路由 URI 添加 `admin` 前缀：

```php
Route::prefix('admin')->group(function () {
    Route::get('users', function () {
        // 匹配 "/admin/users" URL
    });
});
```

### 路由名称前缀

`name` 方法可以用来为组内的每个路由名称添加一个给定的字符串前缀。例如，你可能想使用 `admin` 为所有分组路由的名称添加前缀。给定字符串前缀与指定的路由名称完全相同，因此我们将确保在前缀中提供尾随 `.` 字符：

```php
Route::name('admin.')->group(function () {
    Route::get('users', function () {
        // 路由分配的名称为 "admin.users"...
    })->name('users');
});
```

## 路由模型绑定

当注入一个模型 ID 到一个路由或者控制器动作时，你将经常查询以检索该 ID 对应的模型。Laravel 路由模型绑定提供了一种方便的方式去自动将模型实例直接注入到你的路由。例如，你可以注入与给定 ID 匹配的整个 `User` 模型实例而不是注入一个用户的 ID。

### 隐式绑定

Laravel 自动解析在路由或控制器中定义的 Eloquent 模型，其类型提示的变量名称与路由段名称匹配。例如：

```php
Route::get('api/users/{user}', function (App\User $user) {
    return $user->email;
});
```

由于 `$user` 变量作为 `App\User` Eloquent 模型的类型提示并且变量名称与 `{user}` URI 段相匹配，Laravel 将自动注入模型实例，该模型有一个从请求 URI 的相应值匹配的 ID。如果在数据库中找不到匹配的模型实例，则将自动生成一个 404 HTTP 响应。

#### 自定义键名

当检索一个给定的模型类时，如果你希望模型绑定去使用一个除了 `id` 之外的数据库列，你可以在 Eloquent 模型上覆盖 `getRouteKeyName` 方法：

```php
/**
 * 获取模型的路由键名。
 *
 * @return string
 */
public function getRouteKeyName()
{
    return 'slug';
}
```

### 显式绑定

要注册显示绑定，使用路由的 `model` 方法去指定一个给定参数的类。你应该在 `RouteServiceProvider` 类的 `boot` 方法中定义显示模型绑定：

```php
public function boot()
{
    parent::boot();

    Route::model('user', App\User::class);
}
```

接下来，定义一个包含 `{user}` 参数的路由：

```php
Route::get('profile/{user}', function (App\User $user) {
    //
});
```

由于我们将所有 `{user}` 参数绑定到 `App\User` 模型，一个 `User` 实例将注入到路由。因此，例如，对 `profile/1` 的请求将从 ID 为 `1` 的数据库中注入 `User` 实例。

如果在数据库中找不到匹配的模型实例，一个 404 HTTP 响应将自动生成。

#### 自定义解析逻辑

如果你希望使用你自己的解析逻辑，你可以使用 `Route::bind` 方法。传递给 `bind` 方法的 `Closure` 将接收 URI 段的值，并应当返回应该注入到路由的类的实例：

```php
/**
 * 引导任何应用服务。
 *
 * @return void
 */
public function boot()
{
    parent::boot();

    Route::bind('user', function ($value) {
        return App\User::where('name', $value)->first() ?? abort(404);
    });
}
```

或者，你可以覆盖在 Eloquent 模型上 `resolveRouteBinding` 方法。此方法将接收 URI 段的值，并应当返回应该注入到路由的类的实例：

```php
/**
 * 检索绑定值的模型。
 *
 * @param  mixed  $value
 * @return \Illuminate\Database\Eloquent\Model|null
 */
public function resolveRouteBinding($value)
{
    return $this->where('name', $value)->first() ?? abort(404);
}
```

## 回退路由

使用 `Route::fallback` 方法，你可以定义一个对即将到来的请求没有其它路由匹配时将执行的路由。通常，未处理的请求将通过你的应用程序的异常处理器自动渲染一个 『404』页面。然而，由于你可以在你的 `routes/web.php` 文件中定义 `fallback` 路由，在 `web` 中间件组中的所有中间件将应用到该路由。你根据需要自由地添加额外的中间件到此路由中：

```php
Route::fallback(function () {
    //
});
```

{% hint style="danger" %}

回退路由应该始终是应用程序注册的最后一条路由。

{% endhint %}

## 速率限制

Laravel 在你的应用程序中包含了一个 [中间件](https://laravel.com/docs/5.8/middleware) 对路由进行速率限制访问。首先，将 `throttle` 中间件分配给一个路由或者一组路由。`throttle` 中间件接受两个参数，这两个参数确定在给定的分钟数内能被进行的最大请求数。例如，让我们指定一个认证的用户可以访问如下的路由组每分钟 60 次：

```php
Route::middleware('auth:api', 'throttle:60,1')->group(function () {
    Route::get('/user', function () {
        //
    });
});
```

### 动态速率限制

你可以基于一个认证的 `User` 模型属性指定一个动态请求最大值。例如，如果你的 `User` 模型包含一个 `rate_limit` 属性，你可以传递该属性的名称到 `throttle` 中间件以便于它用于去计算最大的请求数：

```php
Route::middleware('auth:api', 'throttle:rate_limit,1')->group(function () {
    Route::get('/user', function () {
        //
    });
});
```

## 表单方法欺骗

HTML 表单不支持 `PUT`，`PATCH` 或 `DELETE` 动作。因此，当定义从一个 HTML 表单调用的 `PUT`，`PATCH` 或 `DELETE` 路由时，你将需要添加一个隐藏的 `_method` 字段到表单中。用 `_method` 字段发送的值将用于 HTTP 的请求方法：

```php
<form action="/foo/bar" method="POST">
    <input type="hidden" name="_method" value="PUT">
    <input type="hidden" name="_token" value="{{ csrf_token() }}">
</form>
```

你可以使用 `@method` Blade 指令去生成 `_method` 输入：

```php
<form action="/foo/bar" method="POST">
    @method('PUT')
    @csrf
</form>
```

## 访问当前路由

你可以在 `Route` facade 上使用 `current`，`currentRouteName` 和 `currentRouteAction` 方法去访问关于路由处理即将到来的请求的相关信息：

```php
$route = Route::current();

$name = Route::currentRouteName();

$action = Route::currentRouteAction();
```

参考 [路由 facade 基础类](https://laravel.com/api/5.8/Illuminate/Routing/Router.html) 和 [路由实例](https://laravel.com/api/5.8/Illuminate/Routing/Route.html) 的 API 文档，以查看所有可访问的方法。
