# 视图

## 创建视图

{% hint style="info" %}

查找更多关于如何去编写 Blade 模板的信息？查看完整的 [Blade 文档](https://laravel.com/docs/5.8/blade) 去起步。

{% endhint %}

视图包含应用程序提供服务的 HTML，并将你的控制器 / 应用程序逻辑从呈现逻辑中分离。视图存储在 `resources/view` 目录中。一个简单的视图可能看起来像这样：

```php
<!-- 视图存储在 resources/views/greeting.blade.php -->

<html>
    <body>
        <h1>Hello, {{ $name }}</h1>
    </body>
</html>
```

由于此视图存储在 `resources/views/greeting.blade.php`，我们可以使用全局的 `view` 助手函数像这样返回它：

```php
Route::get('/', function () {
    return view('greeting', ['name' => 'James']);
});
```

如你所见，传递到 `view` 助手函数的第一个参数对应于 `resources/views` 目录中视图文件的名称。第二个参数应该是使视图可用的数据数组。在这种情况下，我们传递 `name` 变量，该变量使用 [Blade 语法](https://laravel.com/docs/5.8/blade) 显示在视图中。

视图还可以嵌套在 `resources/views` 目录的子目录中。『点』表示法可以用于引用嵌套的视图。例如，如果视图存储在 `resources/views/admin/profile.blade.php`，你可以像这样引用它：

```php
return view('admin.profile', $data);
```

### 确定一个视图是否存在

如果你需要去确定一个视图是否存在，你可以使用 `View` facade。如果视图存在，`exists` 方法将返回 `true`：

```php
use Illuminate\Support\Facades\View;

if (View::exists('emails.customer')) {
    //
}
```

### 创建第一个可用的视图

使用 `first` 方法，你可以创建存在于一个给定视图数组中的第一个视图。如果你的应用程序或包允许视图自定义或覆盖，这将是有用的：

```php
return view()->first(['custom.admin', 'admin'], $data);
```

你还可以通过 `View` [facade](https://laravel.com/docs/5.8/facades) 调用此方法：

```php
use Illuminate\Support\Facades\View;

return View::first(['custom.admin', 'admin'], $data);
```

## 传递数据到视图

正如你在之前的示例中看到的，你可以传递数据数组到视图：

```php
return view('greetings', ['name' => 'Victoria']);
```

当以这种方式传递信息时，数据应该是具有键 / 值对的数组。在视图中，然后你可以使用对应的键名来访问每个值，比如，`<?php echo $key; ?>`。作为将完整的数据数组传递到 `view` 助手函数的替代，你可以使用 `with` 方法去添加单个数据片段到视图中：

```php
return view('greeting')->with('name', 'Victoria');
```

### 与所有视图共享数据

偶尔，你可能需要与应用程序渲染的所有视图去共享一段数据。你可以使用视图 facade 的 `share` 方法来这样做。通常，你应该在服务提供者的 `boot` 方法中调用 `share`。你随意添加它们到 `AppServiceProvider` 或生成一个单独的服务提供者来安置它们：

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\View;

class AppServiceProvider extends ServiceProvider
{
    /**
     * 引导任何应用程序服务。
     *
     * @return void
     */
    public function boot()
    {
        View::share('key', 'value');
    }

    /**
     * 注册服务提供者。
     *
     * @return void
     */
    public function register()
    {
        //
    }
}
```

## 视图组合器

当一个视图渲染时视图组合器调用回调或类方法。如果你想每次视图渲染时绑定数据到一个视图，则视图组合器能帮助你组织逻辑到单个位置。

对于此示例，让我们用一个 [服务提供者](https://laravel.com/docs/5.8/providers) 注册视图组合器。我们使用 `View` facade 去访问底层的 `Illuminate\Contracts\View\Factory` 合约实现。记住，Laravel 不包含视图组合器的默认目录。按你的希望去随意组织它们。例如，你可以创建一个 `app/Http/View/Composers` 目录：

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\View;
use Illuminate\Support\ServiceProvider;

class ViewServiceProvider extends ServiceProvider
{
    /**
     * 在容器中注册绑定。
     *
     * @return void
     */
    public function boot()
    {
        // 使用基于组合器的类...
        View::composer(
            'profile', 'App\Http\View\Composers\ProfileComposer'
        );

        // 使用基于组合器的 Closure...
        View::composer('dashboard', function ($view) {
            //
        });
    }

    /**
     * 注册服务提供者。
     *
     * @return void
     */
    public function register()
    {
        //
    }
}
```

{% hint style="info" %}

记住，如果你创建一个新的服务提供者去包含你的视图组合器注册，你将需要添加服务提供者到 `config/app.php` 配置文件中的 `providers` 数组。

{% endhint %}

现在我们已注册了组合器，每次 `profile` 视图渲染时 `ProfileComposer@compose` 方法将被执行。所以，让我们定义组合器类：

```php
<?php

namespace App\Http\View\Composers;

use Illuminate\View\View;
use App\Repositories\UserRepository;

class ProfileComposer
{
    /**
     * 用户存储库实现。
     *
     * @var UserRepository
     */
    protected $users;

    /**
     * 创建一个新的个人资料组合器。
     *
     * @param  UserRepository  $users
     * @return void
     */
    public function __construct(UserRepository $users)
    {
        // 服务容器自动解析依赖项...
        $this->users = $users;
    }

    /**
     * 绑定数据到视图。
     *
     * @param  View  $view
     * @return void
     */
    public function compose(View $view)
    {
        $view->with('count', $this->users->count());
    }
}
```

在渲染视图之前，用 `Illuminate\View\View` 实例调用组合器的 `compose` 方法。你可以使用 `with` 方法绑定数据到视图。

{% hint style="info" %}

所有视图组合器通过 [服务容器](https://laravel.com/docs/5.8/container) 解析，因此你可以在组合器的构造方法上键入类型提示你所需的任何依赖项。

{% endhint %}

### 将组合器附加到多个视图

你可以通过传递一个视图数组作为 `composer` 方法的第一个参数，一次附加一个视图组合器到多个视图：

```php
View::composer(
    ['profile', 'dashboard'],
    'App\Http\View\Composers\MyViewComposer'
);
```

`composer` 方法也接受 `*` 字符作为一个通配符，允许你附加一个组合器到所有视图：

```php
View::composer('*', function ($view) {
    //
});
```

### 视图创建器

视图创建器与视图组合器非常相似；然而，它们在视图实例化之后立即执行，而不是等到视图即将渲染之时。要注册一个视图创建器，使用 `creator` 方法：

```php
View::creator('profile', 'App\Http\View\Creators\ProfileCreator');
```
