# 缓存

## 简介

Laravel 为各种缓存后端提供了富有表现力的统一 API。缓存配置位于 `config/cache.php` 中。在此文件中，你可以指定在整个应用程序中默认使用哪个缓存驱动程序。Laravel 支持流行的缓存后端，如：[Memcached](https://memcached.org/) 和 [Redis](https://redis.io/) 开箱即用。

缓存配置文件还包含其他各种选项，这些选项都记录在文件中，所以确保阅读过这些选项。默认情况下，Laravel被配置为使用 `file` 缓存驱动程序，该驱动程序将序列化的缓存对象存储在文件系统中。对于较大的应用程序，建议使用更健壮的驱动程序，如 Memcached 或 Redis。你甚至可以为同一个驱动程序配置多个缓存配置。

### 驱动先决条件

#### 数据库

当使用 `database` 缓存驱动时，你将需要去设置一张表去包含缓存条目。你将发现下面表的一个 `Schema` 声明示例：

```php
Schema::create('cache', function ($table) {
    $table->string('key')->unique();
    $table->text('value');
    $table->integer('expiration');
});
```

{% hint style="info" %}

你也可以使用 `php artisan cache:table` Artisan 命令用合适的模式去生成一个迁移。

{% endhint %}

#### Memcached

使用 Memcached 驱动需要安装 [Memcached PECL package](https://pecl.php.net/package/memcached)。你可以在 `config/cache.php` 配置文件中列出所有 Memcached 服务器：

```php
'memcached' => [
    [
        'host' => '127.0.0.1',
        'port' => 11211,
        'weight' => 100
    ],
],
```

你还可以将 `host` 选项设置为 UNIX 套接字路径。如果这样做，`port` 选项应该设置为 `0`：

```php
'memcached' => [
    [
        'host' => '/var/run/memcached/memcached.sock',
        'port' => 0,
        'weight' => 100
    ],
],
```

#### Redis

在使用 Laravel 的 Redis 缓存之前，你需要通过 Composer 安装 `predis/predis` 软件包（~1.0）或通过 PECL 安装 PhpRedis PHP 扩展。

有关配置 Redis 的更多信息，请参阅其 [Laravel 文档页面](https://laravel.com/docs/5.8/redis#configuration)。

## 缓存用法

### 获取一个缓存实例

`Illuminate\Contracts\Cache\Factory` 和 `Illuminate\Contracts\Cache\Repository contracts` 提供了对 Laravel 缓存服务的访问。`Factory` 合约提供为你的应用程序定义的所有缓存驱动程序的访问。`Repository` 合约通常是你的应用程序的默认缓存驱动程序的实现，通过你的 `cache` 配置文件指定。

不过，你也可以使用 `Cache` 外观，我们将在整个文档中使用缓存外观。`Cache` 外观提供了对 Laravel 缓存合约底层实现的方便、简洁的访问：

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Support\Facades\Cache;

class UserController extends Controller
{
    /**
     * 显示应用程序的所有用户列表。
     *
     * @return Response
     */
    public function index()
    {
        $value = Cache::get('key');

        //
    }
}
```

#### 访问多个缓存存储

使用 `Cache` 外观，你可以通过 `store` 方法访问各种缓存存储。传递给 `store` 方法的键应该对应于 `cache` 配置文件中 `stores` 配置数组中列出的存储驱动之一：

```php
$value = Cache::store('file')->get('foo');

Cache::store('redis')->put('bar', 'baz', 600); // 10 Minutes
```

### 从缓存检索条目

`Cache` 外观上的 `get` 方法用于从缓存中检索条目。如果缓存中不存在条目，则返回 `null`。如果你愿意，可以向 `get` 方法传递第二个参数，指定如果项目不存在，希望返回的默认值：

```php
$value = Cache::get('key');

$value = Cache::get('key', 'default');
```

你甚至可以将 `Closure` 作为默认值传递。如果指定的条目在缓存中不存在，则返回 `Closure` 的结果。传递一个闭包允许你延迟从数据库或其他外部服务检索默认值：

```php
$value = Cache::get('key', function () {
    return DB::table(...)->get();
});
```

#### 检查条目是否存在

`has` 方法可用于确定缓存中是否存在条目。如果值为 `null`，此方法将返回 `false`：

```php
if (Cache::has('key')) {
    //
}
```

#### 递增 / 递减值

`increment` 和 `decrement` 方法可用于调整缓存中整数条目的值。这两个方法都接受一个可选的第二个参数，该参数指示增加或减少条目值的数量：

```php
Cache::increment('key');
Cache::increment('key', $amount);
Cache::decrement('key');
Cache::decrement('key', $amount);
```

#### 检索 & 存储

有时，你可能希望从缓存中检索条目，但如果请求的条目不存在，也要存储一个默认值。例如，你可能希望从缓存中检索所有用户，如果不存在，则从数据库中检索并将其添加到缓存中。你可以使用 `Cache::remember` 方法来完成此操作：

```php
$value = Cache::remember('users', $seconds, function () {
    return DB::table('users')->get();
});
```

如果条目在缓存中不存在，则将执行传递给 `remember` 方法的 `Closure`，并将其结果放在缓存中。

你可以使用 `rememberForever` 方法从缓存或者去检索一个条目或者将其永久存储：

```php
$value = Cache::rememberForever('users', function () {
    return DB::table('users')->get();
});
```

#### 检索 & 删除

如果需要从缓存中检索条目，然后删除条目，可以使用 `pull` 方法。与 `get` 方法一样，如果条目在缓存中不存在，则返回 `null`：

```php
$value = Cache::pull('key');
```

### 在缓存中存储条目

你可以在 `Cache` 外观上使用 `put` 方法在缓存中存储条目：

```php
Cache::put('key', 'value', $seconds);
```

如果存储时间没有传递到 `put` 方法，条目将被无限期存储：

```php
Cache::put('key', 'value');
```

你还可以传递一个表示缓存条目过期时间的 `DateTime` 实例，而不是将秒数作为整数传递：

```php
Cache::put('key', 'value', now()->addMinutes(10));
```

#### 存储如果不存在

如果条目在缓存存储中不存在，`add` 方法只会将其添加到缓存中。如果条目实际添加到缓存中，该方法将返回 `true`。否则，方法将返回 `false`：

```php
Cache::add('key', 'value', $seconds);
```

#### 永远存储条目

`forever` 方法可用于在缓存中永久存储条目。由于这些项不会过期，因此必须使用 `forget` 方法手动从缓存中删除它们：

```php
Cache::forever('key', 'value');
```

{% hint style="info" %}

如果你正在使用 Memcached 驱动程序，当缓存达到其大小限制时，可能会删除『永久』存储的条目。

{% endhint %}

### 从缓存中移除条目

你可以使用 `forget` 方法从缓存中移除条目：

```php
Cache::forget('key');
```

你也可以通过提供一个零或负 TTL 移除条目：

```php
Cache::put('key', 'value', 0);

Cache::put('key', 'value', -5);
```

你可以使用 `flush` 方法清除整个缓存：

```php
Cache::flush();
```

{% hint style="danger" %}

刷新缓存不会考虑缓存前缀，并将从缓存中删除所有条目。当清除其他应用程序共享的缓存时要仔细考虑。

{% endhint %}

### 原子锁

{% hint style="danger" %}

要使用此特性，应用程序必须使用 `memcached`，`dynamodb` 或 `redis` 缓存驱动程序作为你的应用程序的默认缓存驱动程序。此外，所有服务器必须与相同的中央缓存服务器通信。

{% endhint %}

原子锁允许操作分布式锁，而无需担心竞争条件。例如，[Laravel Forge](https://forge.laravel.com/) 使用原子锁来确保一次只在服务器上执行一个远程任务。你可以使用 `Cache::lock` 方法创建和管理锁：

```php
use Illuminate\Support\Facades\Cache;

$lock = Cache::lock('foo', 10);

if ($lock->get()) {
    // Lock acquired for 10 seconds...

    $lock->release();
}
```

`get` 方法也接受一个闭包。在执行闭包之后，Laravel 将自动释放锁：

```php
Cache::lock('foo')->get(function () {
    // 锁定获得无限期和自动释放...
});
```

如果锁在你请求时不可用，你可以指示 Laravel 等待指定的秒数。如果不能在指定的时间限制内获得锁，将抛出一个 `Illuminate\Contracts\Cache\LockTimeoutException` 异常：

```php
use Illuminate\Contracts\Cache\LockTimeoutException;

$lock = Cache::lock('foo', 10);

try {
    $lock->block(5);

    // 等待最多 5 秒后获得锁...
} catch (LockTimeoutException $e) {
    // 不能获得锁...
} finally {
    optional($lock)->release();
}

Cache::lock('foo', 10)->block(5, function () {
    // 等待最多 5 秒后获得锁...
});
```

#### 跨进程管理锁

有时，你可能希望在一个进程中获得一个锁，然后在另一个进程中释放它。例如，你可能在 Web 请求期间获取锁，并希望在由该请求触发的队列作业的末尾释放锁。在此场景中，你应该将锁的作用域『所有者令牌』传递给排队作业，以便作业可以使用给定的令牌重新实例化锁：

```php
// 在控制器内...
$podcast = Podcast::find($id);

$lock = Cache::lock('foo', 120);

if ($result = $lock->get()) {
    ProcessPodcast::dispatch($podcast, $lock->owner());
}

// 在 ProcessPodcast 作业内...
Cache::restoreLock('foo', $this->owner)->release();
```

如果你想在不尊重其当前所有者的情况下释放锁定，可以使用 `forceRelease` 方法：

```php
Cache::lock('foo')->forceRelease();
```

### 缓存助手

除了使用 `Cache` 外观或 [缓存合约](https://laravel.com/docs/5.8/contracts) 外，你还可以使用全局 `cache` 函数来通过缓存检索和存储数据。当使用一个字符串参数调用 `cache` 函数时，它将返回给定键的值：

```php
$value = cache('key');
```

如果提供键 / 值对的数组和到期时间到函数，它将在缓存中存储指定持续时间的值：

```php
cache(['key' => 'value'], $seconds);

cache(['key' => 'value'], now()->addMinutes(10));
```

当在没有任何参数的情况下调用 `cache` 函数时，它将返回一个 `Illuminate\Contracts\Cache\Factory` 实例的实现，从而允许你调用其他缓存方法：

```php
cache()->remember('users', $seconds, function () {
    return DB::table('users')->get();
});
```

{% hint style="info" %}

当测试对全局 `cache` 函数的调用时，你可以使用 `Cache::shouldReceive` 方法，就像你 [测试一个外观](https://laravel.com/docs/5.8/mocking#mocking-facades) 一样。

{% endhint %}

## 缓存标签

{% hint style="danger" %}

使用 `file` 或 `database` 缓存驱动程序时不支持缓存标记。此外，当使用带有『永久』存储的缓存的多个标记时，使用 `memcached` 这样的驱动程序性能最好，因为它会自动清除陈旧的记录。

{% endhint %}

### 存储标记缓存条目

缓存标记允许你标记缓存中的相关条目，然后刷新已分配给给定标记的所有缓存值。你可以通过传入标记名称的有序数组来访问标记缓存。例如，让我们访问带标记的缓存并将值 `put` 到缓存中：

```php
Cache::tags(['people', 'artists'])->put('John', $john, $seconds);

Cache::tags(['people', 'authors'])->put('Anne', $anne, $seconds);
```

### 访问标记的缓存条目

要检索带标记的缓存条目，传递相同的有序标记列表到 `tags` 方法，然后使用你希望检索的键调用 `get` 方法：

```php
$john = Cache::tags(['people', 'artists'])->get('John');

$anne = Cache::tags(['people', 'authors'])->get('Anne');
```

### 移除标记的缓存条目

您可以刷新分配给标记或标记列表的所有条目。例如，该语句将删除所有标记为 `people`，`authors` 或两者都有的缓存。因此，`Anne` 和 `John` 都将从缓存中删除：

```php
Cache::tags(['people', 'authors'])->flush();
```

相反，这个语句只会删除带有 `authors` 标记的缓存，因此将删除 `Anne`，而不会删除 `John`：

```php
Cache::tags('authors')->flush();
```

## 添加自定义缓存驱动

### 编写驱动

要创建自定义缓存驱动程序，首先需要实现 `Illuminate\Contracts\Cache\Store` [合约](https://laravel.com/docs/5.8/contracts)。因此，一个 MongoDB 缓存实现看起来像这样：

```php
<?php

namespace App\Extensions;

use Illuminate\Contracts\Cache\Store;

class MongoStore implements Store
{
    public function get($key) {}
    public function many(array $keys);
    public function put($key, $value, $seconds) {}
    public function putMany(array $values, $seconds);
    public function increment($key, $value = 1) {}
    public function decrement($key, $value = 1) {}
    public function forever($key, $value) {}
    public function forget($key) {}
    public function flush() {}
    public function getPrefix() {}
}
```

我们只需要使用 MongoDB 连接来实现这些中的每个方法。要了解如何实现这些中的每个方法的示例，请查看框架源代码中的 `Illuminate\Cache\MemcachedStore`。一旦实现完成，我们就可以完成自定义驱动程序注册。

```php
Cache::extend('mongo', function ($app) {
    return Cache::repository(new MongoStore);
});
```

{% hint style="info" %}

如果你想知道自定义缓存驱动程序代码放在哪里，可以在你的 `app` 目录中创建一个 `Extensions` 命名空间。但是，记住 Laravel 没有严格的应用程序结构，你可以根据自己的偏好自由组织应用程序。

{% endhint %}

### 注册驱动

要使用 Laravel 注册自定义缓存驱动程序，我们将在 `Cache` 外观上使用 `extend` 方法。对 `Cache::extend` 的调用可以在随新的 Laravel 应用程序一起提供的默认 `App\Providers\AppServiceProvider` 类的 `boot` 方法中完成，或者你可以创建自己的服务提供者来容纳扩展——只是不要忘记在 `config/app.php` 提供者数组中注册自定义的提供者程序：

```php
<?php

namespace App\Providers;

use App\Extensions\MongoStore;
use Illuminate\Support\Facades\Cache;
use Illuminate\Support\ServiceProvider;

class CacheServiceProvider extends ServiceProvider
{
    /**
     * 在容器中注册绑定。
     *
     * @return void
     */
    public function register()
    {
        //
    }

    /**
     * 引导任何应用程序服务。
     *
     * @return void
     */
    public function boot()
    {
        Cache::extend('mongo', function ($app) {
            return Cache::repository(new MongoStore);
        });
    }
}
```

传递给 `extend` 方法的第一个参数是驱动程序的名称。这将与 `config/cache.php` 配置文件中的你的 `driver` 选项相对应。第二个参数是一个Closure，它应该返回一个 `Illuminate\Cache\Repository` 实例。 Closure 将传递一个 `$app` 实例，它是 [服务容器](https://laravel.com/docs/5.8/container) 的一个实例。

一旦你的扩展被注册，将你的 `config/cache.php` 配置文件的 `driver` 选项更新为你的扩展名称。

## 事件

要在每个缓存操作上执行代码，可以侦听缓存触发的 [事件](https://laravel.com/docs/5.8/events)。通常，你应该将这些事件侦听器放置在 `EventServiceProvider` 类中：

```php
/**
 * 应用程序的事件监听器映射。
 *
 * @var array
 */
protected $listen = [
    'Illuminate\Cache\Events\CacheHit' => [
        'App\Listeners\LogCacheHit',
    ],

    'Illuminate\Cache\Events\CacheMissed' => [
        'App\Listeners\LogCacheMissed',
    ],

    'Illuminate\Cache\Events\KeyForgotten' => [
        'App\Listeners\LogKeyForgotten',
    ],

    'Illuminate\Cache\Events\KeyWritten' => [
        'App\Listeners\LogKeyWritten',
    ],
];
```
