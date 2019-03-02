# 服务提供者

* [简介](#jian-jie)
* [编写服务提供者](#bian-xie-fu-wu-qi-gong-zhe)
  * [注册方法](#zhu-ce-fang-fa)
  * [引导方法](#ying-dao-fang-fa)
* [注册提供者](#zhu-ce-qi-gong-zhe)
* [延迟提供者](#yan-chi-qi-gong-zhe)

## 简介

服务提供者是所有 Laravel 应用程序引导的中心地方。你自己的应用程序以及所有 Laravel 的核心服务都是通过服务提供者引导的。

但是，『引导』是什么意思呢？一般来说，我们的意思是 **注册** 事物，包括注册服务容器绑定，事件侦听，中间件甚至路由。服务提供者是配置应用程序的中心地方。

如果你打开 Laravel 包含的 `config/app.php` 文件，你将会看到一个 `providers` 数组。这些所有的服务提供者将由你的应用程序加载。注意其中的许多『延迟』提供者，这意味着它们不会在每个请求中加载，仅当实际需要它们提供的的服务时才加载。

在本概述中，你将学会如何编写你自己的服务提供者并将其注册到 Laravel 应用程序中。

## 编写服务提供者

所有的服务提供者都继承 `Illuminate\Support\ServiceProvider` 类。大部分服务提供者包含一个 `register` 和 `boot` 方法。在 `register` 方法中，你应该 **仅绑定事物到 [服务容器](https://laravel.com/docs/5.8/container)**。你应该从不企图在 `register` 方法中注册任何事件侦听，路由或任何其它功能。

Artisan CLI 通过 `make:provider` 命令可以生成一个新的提供者：

```php
php artisan make:provider RiakServiceProvider
```

### 注册方法

如前所述，在 `register` 方法中，你应该仅绑定事物到 [服务容器](https://laravel.com/docs/5.8/container)。你应该从不企图在 `register` 方法中注册任何事件侦听，路由或任何其它功能。否则，你可能会意外地使用还没有加载的一个提供者提供的服务。

让我们来看一个基本的服务提供者。在任何的服务提供者方法中，你始终可以访问 `$app` 属性，该属性提供对服务容器的访问：

```php
<?php

namespace App\Providers;

use Riak\Connection;
use Illuminate\Support\ServiceProvider;

class RiakServiceProvider extends ServiceProvider
{
    /**
     * 在容器中注册绑定。
     *
     * @return void
     */
    public function register()
    {
        $this->app->singleton(Connection::class, function ($app) {
            return new Connection(config('riak'));
        });
    }
}
```

此服务提供者仅定义一个 `register` 方法并使用在服务容器中该方法定义的一个 `Riak\Connection` 实现。如果你不理解服务容器是如何工作的，查看其 [文档](https://laravel.com/docs/5.8/container)。

#### bindings 和 singletons 属性

如果服务提供者注册了许多简单绑定，你可能希望使用 `bingings` 和 `singletons` 属性而不是手动注册每个服务绑定。当服务提供者被框架加载时，它将自动检查属性和注册它们的绑定：

```php
<?php

namespace App\Providers;

use App\Contracts\ServerProvider;
use App\Contracts\DowntimeNotifier;
use Illuminate\Support\ServiceProvider;
use App\Services\PingdomDowntimeNotifier;
use App\Services\DigitalOceanServerProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * 应该注册的所有容器绑定。
     *
     * @var array
     */
    public $bindings = [
        ServerProvider::class => DigitalOceanServerProvider::class,
    ];

    /**
     * 应该注册的所有容器单例。
     *
     * @var array
     */
    public $singletons = [
        DowntimeNotifier::class => PingdomDowntimeNotifier::class,
    ];
}
```

### 引导方法

因此，如果我们需要去注册一个 [视图 composer](https://laravel.com/docs/5.8/views#view-composers) 在我们的服务提供者该怎么做呢？这应该在 `boot` 方法中做。**所有其它的服务提供者被注册之后调用此方法**，意味着你可以访问所有的被框架注册的其它服务。

```php
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;

class ComposerServiceProvider extends ServiceProvider
{
    /**
     * 引导任何应用程序服务。
     *
     * @return void
     */
    public function boot()
    {
        view()->composer('view', function () {
            //
        });
    }
}
```

#### 引导方法依赖注入

你可以为服务提供者的 `boot` 方法键入类型提示依赖项。[服务容器](https://laravel.com/docs/5.8/container) 将自动注入你需要的任何依赖项：

```php
use Illuminate\Contracts\Routing\ResponseFactory;

public function boot(ResponseFactory $response)
{
    $response->macro('caps', function ($value) {
        //
    });
}
```

## 注册提供者

所有的服务提供者都在 `config/app.php` 配置文件中注册。这个文件包含一个 `providers` 数组，在其中你可以列出服务提供者的类名。默认情况下，在这个数组中列出了一组 Laravel 核心服务提供者。这些提供者引导核心的 Laravel 组件，比如：邮件，队列，缓存其它等。

要注册你的提供者，添加到这个数组：

```php
'providers' => [
    // 其它服务提供者。

    App\Providers\ComposerServiceProvider::class,
],
```

## 延迟提供者

如果你的提供者 **仅** 在 [服务容器](https://laravel.com/docs/5.8/container) 中注册绑定，你可以选择延迟其注册直到实际需要其中一个已注册的绑定。延迟加载这样的一个提供者将提高应用程序的性能，因为它不是在每个请求上从文件系统加载的。

Laravel 编译和存储延迟服务提供者提供的所有服务列表，以及服务提供类的名称。然后，仅当你企图去解析这些服务中的其中一个时，Laravel 才会加载服务提供者。

要延迟加载提供者，实现 `\Illuminate\Contracts\Support\DeferrableProvider` 接口并定义一个 `providers` 方法。`providers` 方法应该通过提供者名称返回服务容器绑定注册的服务：

```php
<?php

namespace App\Providers;

use Riak\Connection;
use Illuminate\Support\ServiceProvider;
use Illuminate\Contracts\Support\DeferrableProvider;

class RiakServiceProvider extends ServiceProvider implements DeferrableProvider
{
    /**
     * 注册服务提供者。
     *
     * @return void
     */
    public function register()
    {
        $this->app->singleton(Connection::class, function ($app) {
            return new Connection($app['config']['riak']);
        });
    }

    /**
     * 通过提供者名称获取服务提供者。
     *
     * @return array
     */
    public function provides()
    {
        return [Connection::class];
    }
}
```
