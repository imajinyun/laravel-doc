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

## 路由参数

## 命名路由

## 路由组

## 路由模型绑定

## 速率限制
