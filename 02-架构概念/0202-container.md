# 服务容器

* [简介](#jian-jie)
* [绑定](#bang-ding)
  * [基础绑定](#ji-chu-bang-ding)
* [解析](#jie-xi)
* [容器事件](#rong-qi-shi-jian)
* [PSR-11](#psr-11)

## 简介

Laravel 服务容器是一个管理类依赖和执行依赖注入的强大工具。依赖注入是一个奇特的短语，本质上意味着：类依赖通过构造方法『注入』到类中，或者在某些情况下，『设置』方法进行注入。

让我们来看一个简单的例子：

```bash
<?php

namespace App\Http\Controllers;

use App\User;
use App\Repositories\UserRepository;
use App\Http\Controllers\Controller;

class UserController extends Controller
{
    /**
     * 用户存储库的实现。
     *
     * @var UserRepository
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

    /**
     * 显示给定用户的介绍。
     *
     * @param  int  $id
     * @return Response
     */
    public function show($id)
    {
        $user = $this->users->find($id);

        return view('user.profile', ['user' => $user]);
    }
}
```

在这个示例中，`UserController` 需要从一个数据源中去检索用户。因此，我们将 **注入** 一个能够检索的用户的服务。在这个上下文中，我们的 `UserRepository` 最有可能使用 [Eloquent](https://laravel.com/docs/5.8/eloquent) 从数据库检索用户信息。但是，由于注入了存储库，我们能够轻松将其与另一个实现交换出来。我们在测试我们的应用程序时还能够轻松『模拟』，或者创建 `UserRepository` 的虚拟实现。

深入理解 Laravel 服务容器对于构建强大的大型应用程序以及为 Laravel 核心本身做贡献是至关重要的。

## 绑定

几乎所有的服务容器绑定将在 [服务提供者](https://laravel.com/docs/5.8/providers) 中注册，因此大多数这些示例将在该上下文中使用容器演示。

{% hint style="info" %}

如果不依赖任何接口就没有必要将类绑定到容器中。容器不需要被指示如果构建这些对象，因此它能使用反射自动解析这些对象。

{% endhint %}

### 基础绑定

#### 简单绑定

在服务容器中，你总是可以通过 `$this->app` 属性访问容器。我们可以使用 `bind` 方法注册一个绑定，传递我们希望去注册的类或接口名称与一个返回一个类实例的 `Closure`：

```php
$this->app->bind('HelpSpot\API', function ($app) {
    return new HelpSpot\API($app->make('HttpClient'));
});
```

注意我们将容器本身作为一个解析器的参数接受。然后我们可以使用容器来解析我们正在构建的对象的子依赖项。

#### 单例绑定

`singleton` 方法绑定一个类或接口到应该仅解析一次的容器中。一旦解析了单例绑定，后续调用容器时将返回相同的对象实例：

```php
$this->app->singleton('HelpSpot\API', function ($app) {
    return new HelpSpot\API($app->make('HttpClient'));
});
```

#### 绑定实例

你还可以使用 `instance` 方法绑定将一个现有的对象实例绑定到容器中。后续调用容器时始终返回给定的给定的实例：

```php
$api = new HelpSpot\API(new HttpClient);

$this->app->instance('HelpSpot\API', $api);
```

#### 绑定参数

有时你可能有一个接受一些注入类的类，但也需要一个注入的原始值如整数。你可以轻松使用上下文绑定来注入类可能需要的任何值：

```php
$this->app->when('App\Http\Controllers\UserController')
          ->needs('$variableName')
          ->give($value);
```

### 接口绑定到实现

服务容器的一个非常强大的功能是它能够将一个接口绑定到一个给定的实现。例如，假设我们有一个 `EventPusher` 接口和一个 `RedisEventPusher` 实现。一旦我们编写了此接口的 `RedisEventPusher` 实现，我们就可以像这样将其注册到服务容器：

```php
$this->app->bind(
    'App\Contracts\EventPusher',
    'App\Services\RedisEventPusher'
);
```

该语句告诉容器当一个类需要一个 `EventPusher` 的实现时它应该注入 `RedisEventPusher`。现在我们可以在构造方法或通过服务容器注入依赖项的任何其它位置键入提示 `EventPusher` 接口：

```php
use App\Contracts\EventPusher;

/**
 * 创建一个新的类实例。
 *
 * @param  EventPusher  $pusher
 * @return void
 */
public function __construct(EventPusher $pusher)
{
    $this->pusher = $pusher;
}
```

### 上下文绑定

有时你可能有两个利用相同接口的类，但你希望在每个类中注入不同的实现。例如，两个控制器可能依赖 `Illuminate\Contracts\Filesystem\Filesystem` [合约](https://laravel.com/docs/5.8/contracts) 的不同实现。Laravel 提供了一个简单，流畅的来定义此行为：

```php
use Illuminate\Support\Facades\Storage;
use App\Http\Controllers\PhotoController;
use App\Http\Controllers\VideoController;
use Illuminate\Contracts\Filesystem\Filesystem;

$this->app->when(PhotoController::class)
          ->needs(Filesystem::class)
          ->give(function () {
              return Storage::disk('local');
          });

$this->app->when([VideoController::class, UploadController::class])
          ->needs(Filesystem::class)
          ->give(function () {
              return Storage::disk('s3');
          });
```

### 标记

偶尔，你可能需要去解决所有绑定的一个确定『分类』。例如，也许你正在构建一个报告聚合器，该聚合器接受许多不同 `Report` 接口实现的数组。注册 `Report` 实现后，你可以使用 `tag` 方法为它们分配一个标记：

```php
$this->app->bind('SpeedReport', function () {
    //
});

$this->app->bind('MemoryReport', function () {
    //
});

$this->app->tag(['SpeedReport', 'MemoryReport'], 'reports');
```

一旦服务被标记了，你可以通过 `tagged` 方法轻松解决他们：

```php
$this->app->bind('ReportAggregator', function ($app) {
    return new ReportAggregator($app->tagged('reports'));
});
```

### 绑定扩展

`extend` 方法允许修改已解析的服务。例如，当一个服务解析后，你可以运行额外的代码去装饰或配置服务。`extend` 方法接口一个闭包，它应该返回修改后的服务，作为唯一的参数：

```php
$this->app->extend(Service::class, function ($service) {
    return new DecoratedService($service);
});
```

## 解析

### make 方法

你可以使用 `make` 方法从容器外去解析一个类。`make` 方法接受你希望解析的类或者接口的名称：

```php
$api = $this->app->make('HelpSpot\API');
```

如果某些类的依赖项无法通过容器解析，你可以通过将它们作为一个关联数组传递到 `makeWith` 方法来注入它们：

```php
$api = $this->app->makeWith('HelpSpot\API', ['id' => 1]);
```

### 自动注入

或者，重要的是，你可以在一个类的构造方法上『类型提示』通过容器解析的依赖项，包括 [控制器](https://laravel.com/docs/5.8/controllers)，[事件侦听](https://laravel.com/docs/5.8/events)，[队列作业](https://laravel.com/docs/5.8/queues)，[中间件](https://laravel.com/docs/5.8/middleware) 等等。实际上，这是通过容器应该解析你的大部分对象的方式。

例如，你可以在构造方法中键入类型提示由你的应用程序定义的存储库。存储库将自动解析并注入到类中：

```php
<?php

namespace App\Http\Controllers;

use App\Users\Repository as UserRepository;

class UserController extends Controller
{
    /**
     * 用户存储实例。
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

    /**
     * 以给定的 ID 展示用户。
     *
     * @param  int  $id
     * @return Response
     */
    public function show($id)
    {
        //
    }
}
```

## 容器事件

服务容器每次解析对象时触发一个事件。你可以使用 `resolving` 方法去侦听这个事件：

```php
$this->app->resolving(function ($object, $app) {
    // 当容器解析任何类型的对象时调用...
});

$this->app->resolving(HelpSpot\API::class, function ($api, $app) {
    // 当容器解析『HelpSpot\API』类型对象时调用...
});
```

如你所见，解析的对象将传递到回调方法，允许你在对象上设置任何额外的属性之后它才被提供给消费者。

## PSR-11

Laravel 的服务容器实例 [PSR-11] 接口。因此，你可以键入类型提示 PSR-11 容器接口以获取 Laravel 容器的一个实例：

```php
use Psr\Container\ContainerInterface;

Route::get('/', function (ContainerInterface $container) {
    $service = $container->get('Service');

    //
});
```

如果给定的标识不能解析，则抛出异常。如果标识从未绑定，则异常将是 `Psr\Container\NotFoundExceptionInterface` 的一个实例。如果标识已绑定但不能解析，将抛出 `Psr\Container\ContainerExceptionInterface` 的一个实例。
