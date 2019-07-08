# 合约

## 简介

Laravel 的合约是一组接口，它定义了通过框架提供的核心服务。例如，一个 `Illuminate\Contracts\Queue\Queue` 合约定义了排队作业所需的方法，而 `Illuminate\Contracts\Mail\Mailer` 合约定义了发送电子邮件所需的方法。

每个合约有一个框架提供的相应实现。例如，Laravel 提供各种驱动程序的队列实现，以及一个由 [SwiftMailer](https://swiftmailer.symfony.com/) 提供支持的邮件程序实现。

所有的 Laravel 合约都存在于 [它们自己的 GitHub 仓库](https://github.com/illuminate/contracts)。这为所有可用的合约以及包开发者可能使用到的单个、解耦的包提供了快速参考点。

### 合约 VS Facades

Laravel 的 [facades](https://laravel.com/docs/5.8/facades) 和助手函数提供一个简单的方式来利用 Laravel 服务，而无需从服务容器中键入类型提示和解析合约。在大多数情况下，每个 facade 都有等效的合约。

与 facades 不同，不要求你在类的构造方法中要求它们，合约允许你为类定义显式的依赖项。一些开发者偏好去显式地定义他们的依赖关系这种方式，并因此更偏爱使用合约，而其他开发人员则喜欢使用 facades。

{% hint style="info" %}

无论你偏爱 facades 或合约，大多数应用程序都可以。但是，如果你构建一个包，你应该强烈地考虑使用合约，因为它们将在包的上下文中很容易测试。

{% endhint %}

## 何时使用合约

正如其它地方所讨论的那样，使用合约还是 Facades 很大程度上取决于你个人品味或开发团队的偏好。合约和 Facades 都可用来构建健壮的、良好测试的 Laravel 应用程序。只要你保持你的类的职责单一，你会发现使用合约和 Facades 的实际差异是很小的。

然而，你仍然可能有几个关于合约的问题。例如，为什么要使用接口？用接口不是更复杂吗？让我们提炼如下标题对于使用接口的原因：松耦合和简单。

### 松耦合

首先，让我们回顾一些与缓存实现紧密耦合的代码。考虑以下几点：

```php
<?php

namespace App\Orders;

class Repository
{
    /**
     * 缓存实例。
     */
    protected $cache;

    /**
     * 创建一个新的存储实例。
     *
     * @param  \SomePackage\Cache\Memcached  $cache
     * @return void
     */
    public function __construct(\SomePackage\Cache\Memcached $cache)
    {
        $this->cache = $cache;
    }

    /**
     * 通过 ID 检索订单。
     *
     * @param  int  $id
     * @return Order
     */
    public function find($id)
    {
        if ($this->cache->has($id))    {
            //
        }
    }
}
```

在这个类中，代码紧密耦合到给定的缓存实现。它紧密耦合是因为我们依赖于包供应商的一个具体缓存类。如果该包的 API 发生改变，我们的代码也必须更改。

同样，如果我们想用另一种的技术（Redis）替换我们的底层缓存技术（Memcached），我们将再次修改我们的存储库。我们的存储库不应该对谁提供数据或者如何提供数据有太多的了解。

**我们可以通过依赖于简单的，与供应商无关的接口来改进我们的代码，而不是之前的方法：**

```php
<?php

namespace App\Orders;

use Illuminate\Contracts\Cache\Repository as Cache;

class Repository
{
    /**
     * 缓存实例。
     */
    protected $cache;

    /**
     * 创建一个新的存储实例。
     *
     * @param  Cache  $cache
     * @return void
     */
    public function __construct(Cache $cache)
    {
        $this->cache = $cache;
    }
}
```

现在代码没有耦合到任何特定的供应商，甚至 Laravel。由于合约包不包含实现和依赖关系，你可以轻松编写任何给定合约的替代实现，允许你去替换你的缓存实现无需修改任何缓存消费代码。

### 简单

当 Laravel 的所有服务都在简单的接口中整齐定义时，很容易通过给定的服务确定提供的功能。**合约允当框架功能的简洁性文档**。

此外，当你依赖简单的接口时，你的代码更容易理解和维护。你可以参考一个简单，干净的接口，而不是在大型复杂的的类中跟踪哪些方法是可用的。

## 如何使用合约

因此，你如何去实现一个合同呢？它实际很简单。

Laravel 中的许多类型的类通过 [服务容器](https://laravel.com/docs/5.8/container) 来解析，包括控制器，事件侦听，中间件，队列作业，甚至路由闭包等。因此，要获取一个合同的实现，你只需在解析的类的构造方法中键入『类型提示』的接口。

例如，看看这个事件侦听器：

```php
<?php

namespace App\Listeners;

use App\User;
use App\Events\OrderWasPlaced;
use Illuminate\Contracts\Redis\Factory;

class CacheOrderInformation
{
    /**
     * Redis 工厂实现。
     */
    protected $redis;

    /**
     * 创建一个新的事件处理器实例。
     *
     * @param  Factory  $redis
     * @return void
     */
    public function __construct(Factory $redis)
    {
        $this->redis = $redis;
    }

    /**
     * 处理事件。
     *
     * @param  OrderWasPlaced  $event
     * @return void
     */
    public function handle(OrderWasPlaced $event)
    {
        //
    }
}
```

当事件侦听器被解析时，服务容器将读取类的构造方法上的类型提示，并注入合适的值。要了解更多有关在服务容器中注册的内容，查看 [服务容器文档](https://laravel.com/docs/5.8/container)。

## 合约参考

此表提供了所有 Laravel 合约及其等效 facades 的一个快速参考：

| Contract                                                                                                                                       | References Facade         |
| ---------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------- |
| [Illuminate\Contracts\Auth\Access\Authorizable](https://github.com/illuminate/contracts/blob/5.8/Auth/Access/Authorizable.php)                 |
| [Illuminate\Contracts\Auth\Access\Gate](https://github.com/illuminate/contracts/blob/5.8/Auth/Access/Gate.php)                                 | `Gate`                    |
| [Illuminate\Contracts\Auth\Authenticatable](https://github.com/illuminate/contracts/blob/5.8/Auth/Authenticatable.php)                         |
| [Illuminate\Contracts\Auth\CanResetPassword](https://github.com/illuminate/contracts/blob/5.8/Auth/CanResetPassword.php)                       |
| [Illuminate\Contracts\Auth\Factory](https://github.com/illuminate/contracts/blob/5.8/Auth/Factory.php)                                         | `Auth`                    |
| [Illuminate\Contracts\Auth\Guard](https://github.com/illuminate/contracts/blob/5.8/Auth/Guard.php)                                             | `Auth::guard()`           |
| [Illuminate\Contracts\Auth\PasswordBroker](https://github.com/illuminate/contracts/blob/5.8/Auth/PasswordBroker.php)                           | `Password::broker()`      |
| [Illuminate\Contracts\Auth\PasswordBrokerFactory](https://github.com/illuminate/contracts/blob/5.8/Auth/PasswordBrokerFactory.php)             | `Password`                |
| [Illuminate\Contracts\Auth\StatefulGuard](https://github.com/illuminate/contracts/blob/5.8/Auth/StatefulGuard.php)                             |
| [Illuminate\Contracts\Auth\SupportsBasicAuth](https://github.com/illuminate/contracts/blob/5.8/Auth/SupportsBasicAuth.php)                     |
| [Illuminate\Contracts\Auth\UserProvider](https://github.com/illuminate/contracts/blob/5.8/Auth/UserProvider.php)                               |
| [Illuminate\Contracts\Bus\Dispatcher](https://github.com/illuminate/contracts/blob/5.8/Bus/Dispatcher.php)                                     | `Bus`                     |
| [Illuminate\Contracts\Bus\QueueingDispatcher](https://github.com/illuminate/contracts/blob/5.8/Bus/QueueingDispatcher.php)                     | `Bus::dispatchToQueue()`  |
| [Illuminate\Contracts\Broadcasting\Factory](https://github.com/illuminate/contracts/blob/5.8/Broadcasting/Factory.php)                         | `Broadcast`               |
| [Illuminate\Contracts\Broadcasting\Broadcaster](https://github.com/illuminate/contracts/blob/5.8/Broadcasting/Broadcaster.php)                 | `Broadcast::connection()` |
| [Illuminate\Contracts\Broadcasting\ShouldBroadcast](https://github.com/illuminate/contracts/blob/5.8/Broadcasting/ShouldBroadcast.php)         |
| [Illuminate\Contracts\Broadcasting\ShouldBroadcastNow](https://github.com/illuminate/contracts/blob/5.8/Broadcasting/ShouldBroadcastNow.php)   |
| [Illuminate\Contracts\Cache\Factory](https://github.com/illuminate/contracts/blob/5.8/Cache/Factory.php)                                       | `Cache`                   |
| [Illuminate\Contracts\Cache\Lock](https://github.com/illuminate/contracts/blob/5.8/Cache/Lock.php)                                             |
| [Illuminate\Contracts\Cache\LockProvider](https://github.com/illuminate/contracts/blob/5.8/Cache/LockProvider.php)                             |
| [Illuminate\Contracts\Cache\Repository](https://github.com/illuminate/contracts/blob/5.8/Cache/Repository.php)                                 | `Cache::driver()`         |
| [Illuminate\Contracts\Cache\Store](https://github.com/illuminate/contracts/blob/5.8/Cache/Store.php)                                           |
| [Illuminate\Contracts\Config\Repository](https://github.com/illuminate/contracts/blob/5.8/Config/Repository.php)                               | `Config`                  |
| [Illuminate\Contracts\Console\Application](https://github.com/illuminate/contracts/blob/5.8/Console/Application.php)                           |
| [Illuminate\Contracts\Console\Kernel](https://github.com/illuminate/contracts/blob/5.8/Console/Kernel.php)                                     | `Artisan`                 |
| [Illuminate\Contracts\Container\Container](https://github.com/illuminate/contracts/blob/5.8/Container/Container.php)                           | `App`                     |
| [Illuminate\Contracts\Cookie\Factory](https://github.com/illuminate/contracts/blob/5.8/Cookie/Factory.php)                                     | `Cookie`                  |
| [Illuminate\Contracts\Cookie\QueueingFactory](https://github.com/illuminate/contracts/blob/5.8/Cookie/QueueingFactory.php)                     | `Cookie::queue()`         |
| [Illuminate\Contracts\Database\ModelIdentifier](https://github.com/illuminate/contracts/blob/5.8/Database/ModelIdentifier.php)                 |
| [Illuminate\Contracts\Debug\ExceptionHandler](https://github.com/illuminate/contracts/blob/5.8/Debug/ExceptionHandler.php)                     |
| [Illuminate\Contracts\Encryption\Encrypter](https://github.com/illuminate/contracts/blob/5.8/Encryption/Encrypter.php)                         | `Crypt`                   |
| [Illuminate\Contracts\Events\Dispatcher](https://github.com/illuminate/contracts/blob/5.8/Events/Dispatcher.php)                               | `Event`                   |
| [Illuminate\Contracts\Filesystem\Cloud](https://github.com/illuminate/contracts/blob/5.8/Filesystem/Cloud.php)                                 | `Storage::cloud()`        |
| [Illuminate\Contracts\Filesystem\Factory](https://github.com/illuminate/contracts/blob/5.8/Filesystem/Factory.php)                             | `Storage`                 |
| [Illuminate\Contracts\Filesystem\Filesystem](https://github.com/illuminate/contracts/blob/5.8/Filesystem/Filesystem.php)                       | `Storage::disk()`         |
| [Illuminate\Contracts\Foundation\Application](https://github.com/illuminate/contracts/blob/5.8/Foundation/Application.php)                     | `App`                     |
| [Illuminate\Contracts\Hashing\Hasher](https://github.com/illuminate/contracts/blob/5.8/Hashing/Hasher.php)                                     | `Hash`                    |
| [Illuminate\Contracts\Http\Kernel](https://github.com/illuminate/contracts/blob/5.8/Http/Kernel.php)                                           |
| [Illuminate\Contracts\Mail\MailQueue](https://github.com/illuminate/contracts/blob/5.8/Mail/MailQueue.php)                                     | `Mail::queue()`           |
| [Illuminate\Contracts\Mail\Mailable](https://github.com/illuminate/contracts/blob/5.8/Mail/Mailable.php)                                       |
| [Illuminate\Contracts\Mail\Mailer](https://github.com/illuminate/contracts/blob/5.8/Mail/Mailer.php)                                           | `Mail`                    |
| [Illuminate\Contracts\Notifications\Dispatcher](https://github.com/illuminate/contracts/blob/5.8/Notifications/Dispatcher.php)                 | `Notification`            |
| [Illuminate\Contracts\Notifications\Factory](https://github.com/illuminate/contracts/blob/5.8/Notifications/Factory.php)                       | `Notification`            |
| [Illuminate\Contracts\Pagination\LengthAwarePaginator](https://github.com/illuminate/contracts/blob/5.8/Pagination/LengthAwarePaginator.php)   |
| [Illuminate\Contracts\Pagination\Paginator](https://github.com/illuminate/contracts/blob/5.8/Pagination/Paginator.php)                         |
| [Illuminate\Contracts\Pipeline\Hub](https://github.com/illuminate/contracts/blob/5.8/Pipeline/Hub.php)                                         |
| [Illuminate\Contracts\Pipeline\Pipeline](https://github.com/illuminate/contracts/blob/5.8/Pipeline/Pipeline.php)                               |
| [Illuminate\Contracts\Queue\EntityResolver](https://github.com/illuminate/contracts/blob/5.8/Queue/EntityResolver.php)                         |
| [Illuminate\Contracts\Queue\Factory](https://github.com/illuminate/contracts/blob/5.8/Queue/Factory.php)                                       | `Queue`                   |
| [Illuminate\Contracts\Queue\Job](https://github.com/illuminate/contracts/blob/5.8/Queue/Job.php)                                               |
| [Illuminate\Contracts\Queue\Monitor](https://github.com/illuminate/contracts/blob/5.8/Queue/Monitor.php)                                       | `Queue`                   |
| [Illuminate\Contracts\Queue\Queue](https://github.com/illuminate/contracts/blob/5.8/Queue/Queue.php)                                           | `Queue::connection()`     |
| [Illuminate\Contracts\Queue\QueueableCollection](https://github.com/illuminate/contracts/blob/5.8/Queue/QueueableCollection.php)               |
| [Illuminate\Contracts\Queue\QueueableEntity](https://github.com/illuminate/contracts/blob/5.8/Queue/QueueableEntity.php)                       |
| [Illuminate\Contracts\Queue\ShouldQueue](https://github.com/illuminate/contracts/blob/5.8/Queue/ShouldQueue.php)                               |
| [Illuminate\Contracts\Redis\Factory](https://github.com/illuminate/contracts/blob/5.8/Redis/Factory.php)                                       | `Redis`                   |
| [Illuminate\Contracts\Routing\BindingRegistrar](https://github.com/illuminate/contracts/blob/5.8/Routing/BindingRegistrar.php)                 | `Route`                   |
| [Illuminate\Contracts\Routing\Registrar](https://github.com/illuminate/contracts/blob/5.8/Routing/Registrar.php)                               | `Route`                   |
| [Illuminate\Contracts\Routing\ResponseFactory](https://github.com/illuminate/contracts/blob/5.8/Routing/ResponseFactory.php)                   | `Response`                |
| [Illuminate\Contracts\Routing\UrlGenerator](https://github.com/illuminate/contracts/blob/5.8/Routing/UrlGenerator.php)                         | `URL`                     |
| [Illuminate\Contracts\Routing\UrlRoutable](https://github.com/illuminate/contracts/blob/5.8/Routing/UrlRoutable.php)                           |
| [Illuminate\Contracts\Session\Session](https://github.com/illuminate/contracts/blob/5.8/Session/Session.php)                                   | `Session::driver()`       |
| [Illuminate\Contracts\Support\Arrayable](https://github.com/illuminate/contracts/blob/5.8/Support/Arrayable.php)                               |
| [Illuminate\Contracts\Support\Htmlable](https://github.com/illuminate/contracts/blob/5.8/Support/Htmlable.php)                                 |
| [Illuminate\Contracts\Support\Jsonable](https://github.com/illuminate/contracts/blob/5.8/Support/Jsonable.php)                                 |
| [Illuminate\Contracts\Support\MessageBag](https://github.com/illuminate/contracts/blob/5.8/Support/MessageBag.php)                             |
| [Illuminate\Contracts\Support\MessageProvider](https://github.com/illuminate/contracts/blob/5.8/Support/MessageProvider.php)                   |
| [Illuminate\Contracts\Support\Renderable](https://github.com/illuminate/contracts/blob/5.8/Support/Renderable.php)                             |
| [Illuminate\Contracts\Support\Responsable](https://github.com/illuminate/contracts/blob/5.8/Support/Responsable.php)                           |
| [Illuminate\Contracts\Translation\Loader](https://github.com/illuminate/contracts/blob/5.8/Translation/Loader.php)                             |
| [Illuminate\Contracts\Translation\Translator](https://github.com/illuminate/contracts/blob/5.8/Translation/Translator.php)                     | `Lang`                    |
| [Illuminate\Contracts\Validation\Factory](https://github.com/illuminate/contracts/blob/5.8/Validation/Factory.php)                             | `Validator`               |
| [Illuminate\Contracts\Validation\ImplicitRule](https://github.com/illuminate/contracts/blob/5.8/Validation/ImplicitRule.php)                   |
| [Illuminate\Contracts\Validation\Rule](https://github.com/illuminate/contracts/blob/5.8/Validation/Rule.php)                                   |
| [Illuminate\Contracts\Validation\ValidatesWhenResolved](https://github.com/illuminate/contracts/blob/5.8/Validation/ValidatesWhenResolved.php) |
| [Illuminate\Contracts\Validation\Validator](https://github.com/illuminate/contracts/blob/5.8/Validation/Validator.php)                         | `Validator::make()`       |
| [Illuminate\Contracts\View\Engine](https://github.com/illuminate/contracts/blob/5.8/View/Engine.php)                                           |
| [Illuminate\Contracts\View\Factory](https://github.com/illuminate/contracts/blob/5.8/View/Factory.php)                                         | `View`                    |
| [Illuminate\Contracts\View\View](https://github.com/illuminate/contracts/blob/5.8/View/View.php)                                               | `View::make()`            |
