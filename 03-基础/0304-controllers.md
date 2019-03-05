# 控制器

## 简介

你可能希望使用控制器类去组织而不是将所有请求处理逻辑定义为路由文件中的 Closures 这种行为。控制器可以将相关的请求处理逻辑组到单个类中。控制器存储在 `app/Http/Controllers` 目录中。

## 基本控制器

### 定义控制器

下面是一个基本控制器类的示例。注意控制器要继承包含在 Laravel 中的基类控制器。基类提供一些方便的方法，例如：`middleware` 方法，它可以被用来附加中间件到控制器动作上：

```php
<?php

namespace App\Http\Controllers;

use App\User;
use App\Http\Controllers\Controller;

class UserController extends Controller
{
    /**
     * 展示给定用户的个人资料。
     *
     * @param  int  $id
     * @return View
     */
    public function show($id)
    {
        return view('user.profile', ['user' => User::findOrFail($id)]);
    }
}
```

你可以像这样定义一个路由到此控制器：

```php
Route::get('user/{id}', 'UserController@show');
```

现在，当请求匹配指定的路由 URI，在 `UserController` 类上的 `show` 方法将被执行。路由参数也将被传递到方法中：

{% hint style="info" %}

控制器不 **强制** 去继承一个基类。然而，你将不能方便地去访问如 `middleware`，`validate` 和 `dispatch` 等功能方法。

{% endhint %}

### 控制器 & 命名空间

当定义控制器路由时，注意我们不需要去指定完整的控制器命名空间是重要的。由于 `RouteServiceProvider` 使用包含命名空间的路由组加载你的路由文件，因此我们仅指定跟随在命名空间 `App\Http\Controllers` 部分之后的类名部分。

如果你选择去嵌套你的控制到 `App\Http\Controllers` 目录的更深出，使用相对于 `App\Http\Controllers` 根命名空间的特定类名。因此，如果你的完整的控制器类是 `App\Http\Controllers\Photos\AdminController`，你应该像这样注册路由到控制器中：

```php
Route::get('foo', 'Photos\AdminController@method');
```

### 单个动作控制器

如果你希望去定义一个仅处理单个动作的控制器，你可以在控制器中放置一个 `__invoke` 方法：

```php
<?php

namespace App\Http\Controllers;

use App\User;
use App\Http\Controllers\Controller;

class ShowProfile extends Controller
{
    /**
     * 展示给定用户的个人资料。
     *
     * @param  int  $id
     * @return View
     */
    public function __invoke($id)
    {
        return view('user.profile', ['user' => User::findOrFail($id)]);
    }
}
```

当为单个动作的控制器注册路由时，你无需去指定一个方法：

```php
Route::get('user/{id}', 'ShowProfile');
```

你可以使用 `make:controller` Artisan 命令的 `--invokable` 选项去生成一个可调用的控制器：

```php
php artisan make:controller ShowProfile --invokable
```

## 控制器中间件

## 资源控制器

### 部分资源路由

### 命名资源路由

### 命名资源路由参数

### 本地化资源 URIs

### 补充资源控制器

## 依赖注入 & 控制器

## 路由缓存
