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

### 生成指定路由的 URL

### 检索当前路由

## 路由组

## 路由模型绑定

## 速率限制
