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

在你的路由文件中可以把 [中间件](https://laravel.com/docs/5.8/middleware) 分配到控制器的路由上：

```bash
Route::get('profile', 'UserController@show')->middleware('auth');
```

然而，在你的控制器的构造方法中指定中间件是更方便。从你的控制器的构造方法中使用 `middleware` 方法，你可以轻松分配中间到控制器的动作。你甚至可以将中间件限定为仅在控制器类的某些方法上：

```php
class UserController extends Controller
{
    /**
     * 实例化一个新的控制器实例。
     *
     * @return void
     */
    public function __construct()
    {
        $this->middleware('auth');

        $this->middleware('log')->only('index');

        $this->middleware('subscribed')->except('store');
    }
}
```

控制器还允许你使用 Closure 去注册中间件。这为定义单个控制器的中间件提供了一个方便的方法，而无需定义整个中间件类：

```php
$this->middleware(function ($request, $next) {
    // ...

    return $next($request);
});
```

{% hint style="danger" %}

你可以将中间件分配给控制器的子集；然而，它可能预示着你的控制器增长过大。相反，考虑将控制器分成多个较小的控制器。

{% endhint %}

## 资源控制器

Laravel 资源路由将典型的『CRUD』路由分配到具有单行代码的一个控制器上。例如，你可能希望创建一个控制器来处理应用程序存储的『照片』的所有 HTTP 请求。使用 `make:controller` Artisan 命令，我们可以快速创建这个一个控制器：

```php
php artisan make:controller PhotoController --resource
```

此命令将生成一个控制器位于 `app/Http/Controllers/PhotoController.php`。控制器将包含为每个可用资源操作的一个方法。

接下来，你可以向控制器注册一个资源路由：

```php
Route::resource('photos', 'PhotoController');
```

此单一路由声明创建了多个路由去处理资源上的各种操作。生成的控制器已经有针对每个动作的预留方法，包括通知你它们处理的 HTTP 动词和 URI 的注释。

你可以通过传递一个数组到 `resources` 方法来一次注册许多资源控制器：

```php
Route::resources([
    'photos' => 'PhotoController',
    'posts' => 'PostController'
]);
```

**资源控制器动作处理**

| 动词        | URI                    | 动作    | 路由名称       |
| ----------- | ---------------------- | ------- | -------------- |
| `GET`       | `/photos`              | index   | photos.index   |
| `GET`       | `/photos/create`       | create  | photos.create  |
| `POST`      | `/photos`              | store   | photos.store   |
| `GET`       | `/photos/{photo}`      | show    | photos.show    |
| `GET`       | `/photos/{photo}/edit` | edit    | photos.edit    |
| `PUT/PATCH` | `/photos/{photo}`      | update  | photos.update  |
| `DELETE`    | `/photos/{photo}`      | destroy | photos.destroy |

**指定资源模型**

如你使用路由模型绑定并希望资源控制器的方法键入提示模型实例，你可以在生成控制器时使用 `--model` 选项：

```bash
php artisan make:controller PhotoController --resource --model=Photo
```

**欺骗表单方法**

由于 HTML 表单不能发送 `PUT`，`PATCH` 或 `DELETE` 请求，你将需要去添加一个隐藏的 `_method` 字段去欺骗这些 HTTP 动词。`@method` Blade 指令可以为你创建此字段：

```php
<form action="/foo/bar" method="POST">
    @method('PUT')
</form>
```

### 部分资源路由

当声明一个资源路由时，你可以指定指定控制器应处理的动作子集而不是完整的默认动作集：

```php
Route::resource('photos', 'PhotoController')->only([
    'index', 'show'
]);

Route::resource('photos', 'PhotoController')->except([
    'create', 'store', 'update', 'destroy'
]);
```

#### API 资源路由

当声明通过 API 使用的资源路由时，通常你将需要排除呈现 HTML 模板的路由，比如：`create` 和 `edit`。为了方便起见，你可以使用 `apiResource` 方法去自动排除这两个路由：

```php
Route::apiResource('photos', 'PhotoController');
```

你可以通过传递一个数组到 `apiResources` 方法来一次注册许多 API 资源控制器：

```php
Route::apiResources([
    'photos' => 'PhotoController',
    'posts' => 'PostController'
]);
```

为了快速生成一个 API 资源控制器而不包括 `create` 或 `edit` 方法，在执行 `make:controller` 命令时使用 `--api` 选项：

```php
php artisan make:controller API/PhotoController --api
```

### 命名资源路由

默认情况下，所有资源控制器动作有一个路由名称；然而，你可以通过使用你的选项传递一个 `names` 数组覆盖这些名称：

```php
Route::resource('photos', 'PhotoController')->names([
    'create' => 'photos.build'
]);
```

### 命名资源路由参数

默认情况下，`Route::resource` 将基于资源名称的『单数化』版本为你的资源路由创建路由参数。你可以使用 `parameters` 方法在每个资源的基础上轻松覆盖它。传递给 `parameters` 方法的数组应该是一个资源名称和参数名称关联的数组：

```php
Route::resource('users', 'AdminUserController')->parameters([
    'users' => 'admin_user'
]);
```

以上的示例为资源的 `show` 路由生成如下的 URI：

```bash
/users/{admin_user}
```

### 本地化资源 URIs

默认情况下，`Route::resource` 将使用英文动词创建资源 URI。如果你需要本地化 `create` 和 `edit` 动作动词，你可以使用 `Route::resourceVerbs` 方法。在你的 `AppServiceProvider` 类的 `boot` 方法可以这样做：

```php
use Illuminate\Support\Facades\Route;

/**
 * 引导任何应用程序服务。
 *
 * @return void
 */
public function boot()
{
    Route::resourceVerbs([
        'create' => 'crear',
        'edit' => 'editar',
    ]);
}
```

一旦动词被自定义，一个资源路由注册如 `Route::resource('fotos', 'PhotoController')` 将产生如下的 URI：

```bash
/fotos/crear

/fotos/{foto}/editar
```

### 补充资源控制器

如果你需要在超出默认资源路由集的一个资源控制器中添加额外的路由，你应该调用 `Route::resource` 之前定义这些路由；否则，通过 `resource` 方法定义的路由可能会无意中优先于你的补充路由：

```php
Route::get('photos/popular', 'PhotoController@method');

Route::resource('photos', 'PhotoController');
```

{% hint style="info" %}

记住要保持你的控制器聚集，如果你发现自己经常需要需要典型的资源动作集之外的方法，考虑将控制器分隔成两个更小的控制器。

{% endhint %}

## 依赖注入 & 控制器

### 构造方法注入

Laravel [服务容器](https://laravel.com/docs/5.8/container) 被用来解析所有的 Laravel 控制器。正应如此，你可以在构造方法中键入类型提示你的控制器需要的任何依赖项。声明依赖项将自动解析和注入到控制器实例上：

```php
<?php

namespace App\Http\Controllers;

use App\Repositories\UserRepository;

class UserController extends Controller
{
    /**
     * 用户存储库实例。
     */
    protected $users;

    /**
     * 创建一个新的控制器实例。
     *
     * @param  UserRepository  $users
     * @return void
     */
    public function __construct(UserRepository $users)
    {
        $this->users = $users;
    }
}
```

你还可以键入任何 [Laravel 合约](https://laravel.com/docs/5.8/contracts) 类型提示。如果容器能解析它，你可以键入类型提示。根据于你的应用程序，注入依赖项到你的控制器可能提供更好的可测试性。

### 方法注入

除了构造方法注入之外，你还可以在你的控制器的方法上键入类型提示依赖项。方法注入的一个常用情形是将 `Illuminate\Http\Request` 实例注入到你的控制器方法中：

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
        $name = $request->name;

        //
    }
}
```

如果你的控制器方法也期望从路由参数中输入，在其它依赖之后列出你的路由参数。例如，如果你的路由像这样定义：

```php
Route::put('user/{id}', 'UserController@update');
```

你仍然可以通过定义你的控制器方法键入类型提示 `Illuminate\Http\Request` 并访问你的 `id` 参数，如下所示：

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class UserController extends Controller
{
    /**
     * 更新给定的用户。
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

## 路由缓存

{% hint style="danger" %}

基于 Closure 的路由不能被缓存。要使用路由缓存，你必须将任何 Closure 路由转换为控制器路由。

{% endhint %}

如果你的应用程序专门使用基于控制器的路由，你应该利用 Laravel 的路由缓存。使用路由缓存将大幅减少注册应用程序所有路由所需的时间。在某些情况下，你的路由注册速度基于可能会提高 100 倍。要生成一个路由缓存，仅需要执行 `route:cache` Artisan 命令：

```php
php artisan route:cache
```

运行此命令之后，将在你的每个请求上加载缓存的路由文件。记住，如果你添加任何新的路由，你需要去生成一个新的路由缓存。正因如此，你应该仅在你的项目部署期间运行 `route:cache` 命令。

你可以使用 `route:clear` 命令去清理路由缓存：

```php
php artisan route:clear
```
