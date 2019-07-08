# Facades

## 简介

Facades 在应用程序服务容器中可用的类提供了一个『静态』接口。Laravel 拥有许多 facades，它提供了几乎所有访问 Laravel 的功能。Laravel facades 充当服务容器中底层类的『静态代理』，提供简洁，表达性强的语法，同时比传统静态类方法维护更加可测试和灵活。

Laravel 所有的 facades 都在 `Illuminate\Support\Facades` 命名空间中定义。因此，我们可以像这样轻松访问 facades：

```php
use Illuminate\Support\Facades\Cache;

Route::get('/cache', function () {
    return Cache::get('key');
});
```

纵观 Laravel 文档，许多实例将使用 facades 去演示框架的各种功能。

## 何时使用 Facades

Facades 有很多好处。它们提供简洁，令人难忘的语法，允许你使用 Laravel 的功能而不需记住必须手动注入和配置的长类名。此外，因为它们独特的使用 PHP 的动态方法，很容易测试。

然而，使用 facades 时必须小心。facades 最主要的危险是引起类作用范围的蔓延。由于 facades 是如此容易的使用并不需要注入，它可以很容易让你的类持续增长并在单个类中使用许多 facades。使用依赖注入，这种潜在的问题可以通过一个大型构造方法以及类增长过大的视觉反馈得到缓解。因此，当使用 facades 时，特别注意你的类规模以便使其责任范围不会太大。

{% hint style="info" %}

在构建与 Laravel 进行交互的第三方包时，最好选择是注入 [Laravel 合约](https://laravel.com/docs/5.8/contracts) 而不使用 facades。因为第三方包是在 Laravel 自身之外构建的，你将无法访问 Laravel 的 facades 测试辅助函数。

{% endhint %}

### Facades VS 依赖注入

依赖注入的主要好处之一是它能够交换注入类的实现。在测试期间非常有用，因为你可以注入 mock 或 stub 并断言在 stub 上调用的各种方法。

通常，不可能去 mock 或 stub 一个真实的静态类方法。但是，由于 facades 使用动态方法去代理从服务容器解析的对象的方法调用，我们实际上可以测试 facades 就像我们测试一个注入的类实例一样。例如，给出如下的路由：

```php
use Illuminate\Support\Facades\Cache;

Route::get('/cache', function () {
    return Cache::get('key');
});
```

我们可以编写如下的测试去验证 `Cache::get` 方法以我们期望的参数被调用：

```php
use Illuminate\Support\Facades\Cache;

/**
 * 一个基本的功能测试示例。
 *
 * @return void
 */
public function testBasicExample()
{
    Cache::shouldReceive('get')
         ->with('key')
         ->andReturn('value');

    $this->visit('/cache')
         ->see('value');
}
```

### Facades VS 助手函数

除了 facades，Laravel 包含各种『助手』函数能够执行通用的任务，像生成视图，触发事件，调度作业或发送 HTTP 响应。许多这些助手函数执行与相应 facade 相同的功能。例如，此 facades 调用和助手调用等效：

```php
return View::make('profile');

return view('profile');
```

Facades 与助手函数之间完全没有实际区别。当使用助手函数时，你仍然可以像测试相应的 facade 一样精确地测试它们。例如，给定如下的路由：

```php
Route::get('/cache', function () {
    return cache('key');
});
```

在底层，`cache` 助手将调用 `Cache` facade 下的类的 `get` 方法。因此，即使我们使用助手方法，我们可以编写如下的测试去验证此方法以我们期望的参数被调用：

```php
use Illuminate\Support\Facades\Cache;

/**
 * 一个基本的功能测试示例。
 *
 * @return void
 */
public function testBasicExample()
{
    Cache::shouldReceive('get')
         ->with('key')
         ->andReturn('value');

    $this->visit('/cache')
         ->see('value');
}
```

## Facades 如何工作

在 Laravel 应用程序中，一个 facade 是一个提供从容器中访问对象的类。这种机制使这项工作在 `Facade` 类中。Laravel 的 facades，并且你创建的任何自定义 facades，将继承 `Illuminate\Support\Facades\Facade` 基类。

`Facade` 基类使用 `__callStatic()` 魔术方法将你编写的 facade 从容器中解析的一个对象中延迟调用。通过浏览此代码，可以假设在 `Cache` 类上调用静态的方法 `get`：

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use Illuminate\Support\Facades\Cache;

class UserController extends Controller
{
    /**
     * 显示给定用户的介绍。
     *
     * @param  int  $id
     * @return Response
     */
    public function showProfile($id)
    {
        $user = Cache::get('user:'.$id);

        return view('profile', ['user' => $user]);
    }
}
```

注意文件顶部附近，我们『导入』`Cache` facade。此 facade 充当代理去访问 `Illuminate\Contracts\Cache\Factory` 接口的底层实现。我们使用 Facade 进行的任何调用将传递到 Laravel 的缓存服务的底层实现上。

如果浏览 `Illuminate\Support\Facades\Cache` 类，将看到没有静态方法 `get`：

```php
class Cache extends Facade
{
    /**
     * 获取组件的注册名称。
     *
     * @return string
     */
    protected static function getFacadeAccessor() { return 'cache'; }
}
```

相反，`Cache` facade 继承基类 `Facade` 类并定义方法 `getFacadeAccessor()`。此方法的作用是返回服务容器绑定的名称。当一个用户引用 `Cache` facade 上的任何静态方法时，Laravel 从 [服务容器](https://laravel.com/docs/5.8/container) 中解析 `cache` 绑定并运行该对象所请求的方法（在本例中为 `get`）。

## 实时 Facades

使用实时 facades，你可以将应用程序中的任何类视为 facade。为了说明如何使用它，让我们来看一个替代方案。例如，假设我们的 `Podcast` 模型有一个 `publish` 模型。但是，为了发布播客，我们需要去注入一个 `Publisher` 实例：

```php
<?php

namespace App;

use App\Contracts\Publisher;
use Illuminate\Database\Eloquent\Model;

class Podcast extends Model
{
    /**
     * 发布播客。
     *
     * @param  Publisher  $publisher
     * @return void
     */
    public function publish(Publisher $publisher)
    {
        $this->update(['publishing' => now()]);

        $publisher->publish($this);
    }
}
```

注入一个发布者实现到方法中将允许我们轻松地单独测试该方法，因为我们可以 mock 注入的发布者。但是，它要求我们每次调用 `publish` 方法时始终传递一个发布者实例。使用实时 facades，我们可以维护相同的可测试性，同时不需要显式传递一个 `Publisher` 实例。要生成一个实时 facade，在导入类的名称空间中前加上 `Facades` 前缀：

```php
<?php

namespace App;

use Facades\App\Contracts\Publisher;
use Illuminate\Database\Eloquent\Model;

class Podcast extends Model
{
    /**
     * 发布播客。
     *
     * @return void
     */
    public function publish()
    {
        $this->update(['publishing' => now()]);

        Publisher::publish($this);
    }
}
```

当使用实时外观时，发布者实现将使用出现在 `Facades` 前缀之后的接口部分或类名从服务容器中解析出来。在测试时，我们可以使用 Laravel 的内置 facade 测试助手去 mock 这个方法调用：

```php
<?php

namespace Tests\Feature;

use App\Podcast;
use Tests\TestCase;
use Facades\App\Contracts\Publisher;
use Illuminate\Foundation\Testing\RefreshDatabase;

class PodcastTest extends TestCase
{
    use RefreshDatabase;

    /**
     * 一个测试实例。
     *
     * @return void
     */
    public function test_podcast_can_be_published()
    {
        $podcast = factory(Podcast::class)->create();

        Publisher::shouldReceive('publish')->once()->with($podcast);

        $podcast->publish();
    }
}
```

## Facades 类参考

你将在下面找到每个 facade 类及其底层类。这是一个有用的工具，可以快速寻找给定 facade 类的 API 文档。[服务容器绑定](https://laravel.com/docs/5.8/container) 的关键信息也包含在内。

| Facade               | Class                                                                                                                               | Service Container Binding |
| -------------------- | ----------------------------------------------------------------------------------------------------------------------------------- | ------------------------- |
| App                  | [Illuminate\Foundation\Application](https://laravel.com/api/5.8/Illuminate/Foundation/Application.html)                             | `app`                     |
| Artisan              | [Illuminate\Contracts\Console\Kernel](https://laravel.com/api/5.8/Illuminate/Contracts/Console/Kernel.html)                         | `artisan`                 |
| Auth                 | [Illuminate\Auth\AuthManager](https://laravel.com/api/5.8/Illuminate/Auth/AuthManager.html)                                         | `auth`                    |
| Auth (Instance)      | [Illuminate\Contracts\Auth\Guard](https://laravel.com/api/5.8/Illuminate/Contracts/Auth/Guard.html)                                 | `auth.driver`             |
| Blade                | [Illuminate\View\Compilers\BladeCompiler](https://laravel.com/api/5.8/Illuminate/View/Compilers/BladeCompiler.html)                 | `blade.compiler`          |
| Broadcast            | [Illuminate\Contracts\Broadcasting\Factory](https://laravel.com/api/5.8/Illuminate/Contracts/Broadcasting/Factory.html)             |
| Broadcast (Instance) | [Illuminate\Contracts\Broadcasting\Broadcaster](https://laravel.com/api/5.8/Illuminate/Contracts/Broadcasting/Broadcaster.html)     |
| Bus                  | [Illuminate\Contracts\Bus\Dispatcher](https://laravel.com/api/5.8/Illuminate/Contracts/Bus/Dispatcher.html)                         |
| Cache                | [Illuminate\Cache\CacheManager](https://laravel.com/api/5.8/Illuminate/Cache/CacheManager.html)                                     | `cache`                   |
| Cache (Instance)     | [Illuminate\Cache\Repository](https://laravel.com/api/5.8/Illuminate/Cache/Repository.html)                                         | `cache.store`             |
| Config               | [Illuminate\Config\Repository](https://laravel.com/api/5.8/Illuminate/Config/Repository.html)                                       | `config`                  |
| Cookie               | [Illuminate\Cookie\CookieJar](https://laravel.com/api/5.8/Illuminate/Cookie/CookieJar.html)                                         | `cookie`                  |
| Crypt                | [Illuminate\Encryption\Encrypter](https://laravel.com/api/5.8/Illuminate/Encryption/Encrypter.html)                                 | `encrypter`               |
| DB                   | [Illuminate\Database\DatabaseManager](https://laravel.com/api/5.8/Illuminate/Database/DatabaseManager.html)                         | `db`                      |
| DB (Instance)        | [Illuminate\Database\Connection](https://laravel.com/api/5.8/Illuminate/Database/Connection.html)                                   | `db.connection`           |
| Event                | [Illuminate\Events\Dispatcher](https://laravel.com/api/5.8/Illuminate/Events/Dispatcher.html)                                       | `events`                  |
| File                 | [Illuminate\Filesystem\Filesystem](https://laravel.com/api/5.8/Illuminate/Filesystem/Filesystem.html)                               | `files`                   |
| Gate                 | [Illuminate\Contracts\Auth\Access\Gate](https://laravel.com/api/5.8/Illuminate/Contracts/Auth/Access/Gate.html)                     |
| Hash                 | [Illuminate\Contracts\Hashing\Hasher](https://laravel.com/api/5.8/Illuminate/Contracts/Hashing/Hasher.html)                         | `hash`                    |
| Lang                 | [Illuminate\Translation\Translator](https://laravel.com/api/5.8/Illuminate/Translation/Translator.html)                             | `translator`              |
| Log                  | [Illuminate\Log\LogManager](https://laravel.com/api/5.8/Illuminate/Log/LogManager.html)                                             | `log`                     |
| Mail                 | [Illuminate\Mail\Mailer](https://laravel.com/api/5.8/Illuminate/Mail/Mailer.html)                                                   | `mailer`                  |
| Notification         | [Illuminate\Notifications\ChannelManager](https://laravel.com/api/5.8/Illuminate/Notifications/ChannelManager.html)                 |
| Password             | [Illuminate\Auth\Passwords\PasswordBrokerManager](https://laravel.com/api/5.8/Illuminate/Auth/Passwords/PasswordBrokerManager.html) | `auth.password`           |
| Password (Instance)  | [Illuminate\Auth\Passwords\PasswordBroker](https://laravel.com/api/5.8/Illuminate/Auth/Passwords/PasswordBroker.html)               | `auth.password.broker`    |
| Queue                | [Illuminate\Queue\QueueManager](https://laravel.com/api/5.8/Illuminate/Queue/QueueManager.html)                                     | `queue`                   |
| Queue (Instance)     | [Illuminate\Contracts\Queue\Queue](https://laravel.com/api/5.8/Illuminate/Contracts/Queue/Queue.html)                               | `queue.connection`        |
| Queue (Base Class)   | [Illuminate\Queue\Queue](https://laravel.com/api/5.8/Illuminate/Queue/Queue.html)                                                   |
| Redirect             | [Illuminate\Routing\Redirector](https://laravel.com/api/5.8/Illuminate/Routing/Redirector.html)                                     | `redirect`                |
| Redis                | [Illuminate\Redis\RedisManager](https://laravel.com/api/5.8/Illuminate/Redis/RedisManager.html)                                     | `redis`                   |
| Redis (Instance)     | [Illuminate\Redis\Connections\Connection](https://laravel.com/api/5.8/Illuminate/Redis/Connections/Connection.html)                 | `redis.connection`        |
| Request              | [Illuminate\Http\Request](https://laravel.com/api/5.8/Illuminate/Http/Request.html)                                                 | `request`                 |
| Response             | [Illuminate\Contracts\Routing\ResponseFactory](https://laravel.com/api/5.8/Illuminate/Contracts/Routing/ResponseFactory.html)       |
| Response (Instance)  | [Illuminate\Http\Response](https://laravel.com/api/5.8/Illuminate/Http/Response.html)                                               |
| Route                | [Illuminate\Routing\Router](https://laravel.com/api/5.8/Illuminate/Routing/Router.html)                                             | `router`                  |
| Schema               | [Illuminate\Database\Schema\Builder](https://laravel.com/api/5.8/Illuminate/Database/Schema/Builder.html)                           |
| Session              | [Illuminate\Session\SessionManager](https://laravel.com/api/5.8/Illuminate/Session/SessionManager.html)                             | `session`                 |
| Session (Instance)   | [Illuminate\Session\Store](https://laravel.com/api/5.8/Illuminate/Session/Store.html)                                               | `session.store`           |
| Storage              | [Illuminate\Filesystem\FilesystemManager](https://laravel.com/api/5.8/Illuminate/Filesystem/FilesystemManager.html)                 | `filesystem`              |
| Storage (Instance)   | [Illuminate\Contracts\Filesystem\Filesystem](https://laravel.com/api/5.8/Illuminate/Contracts/Filesystem/Filesystem.html)           | `filesystem.disk`         |
| URL                  | [Illuminate\Routing\UrlGenerator](https://laravel.com/api/5.8/Illuminate/Routing/UrlGenerator.html)                                 | `url`                     |
| Validator            | [Illuminate\Validation\Factory](https://laravel.com/api/5.8/Illuminate/Validation/Factory.html)                                     | `validator`               |
| Validator (Instance) | [Illuminate\Validation\Validator](https://laravel.com/api/5.8/Illuminate/Validation/Validator.html)                                 |
| View                 | [Illuminate\View\Factory](https://laravel.com/api/5.8/Illuminate/View/Factory.html)                                                 | `view`                    |
| View (Instance)      | [Illuminate\View\View](https://laravel.com/api/5.8/Illuminate/View/View.html)                                                       |
