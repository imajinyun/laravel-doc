# 事件

## 简介

Laravel 的事件提供了一个简单的观察器实现，允许你订阅和监听应用程序中发生的各种事件。事件类通常存储在 `app/Events` 目录中，而他们的监听器存储在 `app/Listeners` 中。如果你在应用程序中没有看到这些目录，请不要担心，因为在你使用 Artisan 控制台命令生成事件和监听器时将为你创建这些目录。

事件是一种很好的方法来解耦应用程序的各个方面，因为一个事件可以有多个互不依赖的监听器。例如，你可能希望在每次发送订单时向用户发送一个 Slack 通知。你可以引发 `OrderShipped` 事件，监听器可以接收该事件并将其转换为 Slack 通知，而不是将订单处理代码与 Slack 通知代码耦合。

## 注册事件 & 监听器

Laravel 应用程序中包含的 `EventServiceProvider` 提供了一个方便的位置来注册应用程序的所有事件监听器。`listen` 属性包含所有事件（键）及其监听器（值）的数组。您可以根据应用程序的需要向这个数组添加任意数量的事件。例如，让我们添加一个 `OrderShipped` 事件：

```php
/**
 * The event listener mappings for the application.
 *
 * @var array
 */
protected $listen = [
    'App\Events\OrderShipped' => [
        'App\Listeners\SendShipmentNotification',
    ],
];
```

### 生成事件 & 监听

当然，手动为每个事件和监听器创建文件是很麻烦的。相反，将监听器和事件添加到 `EventServiceProvider` 并使用 `event:generate` 命令。此命令将生成 `EventServiceProvider` 中列出的任何事件或监听器。已经存在的事件和监听器将保持不变：

```bash
php artisan event:generate
```

### 手动注册事件

通常，事件应该通过 `EventServiceProvider` `$listen` 数组注册；不过，你也可以在 `EventServiceProvider` 的 `boot` 方法中手动注册基于闭包的事件：

```php
/**
 * Register any other events for your application.
 *
 * @return void
 */
public function boot()
{
    parent::boot();

    Event::listen('event.name', function ($foo, $bar) {
        //
    });
}
```

#### 通配符事件监听器

你甚至可以使用 `*` 作为通配符参数注册监听器，从而允许你在同一个监听器上捕获多个事件。通配符监听器接收事件名称作为第一个参数，整个事件数据数组作为第二个参数：

```php
Event::listen('event.*', function ($eventName, array $data) {
    //
});
```

### 事件发现

{% hint style="danger" %}

事件发现在 Laravel 5.8.9 或以后的版本中可用。

{% endhint %}

你可以启用自动事件发现，而不是在 `EventServiceProvider` 的 `$listen` 数组中手动注册事件和监听器。启用事件发现后，Laravel 将通过扫描应用程序的 `Listeners` 目录自动查找和注册事件和监听器。此外，`EventServiceProvider` 中列出的任何显式定义的事件仍然会被注册。

Laravel 通过使用反射扫描监听器类来查找事件监听器。当 Laravel 找到任何以 `handle` 开头的监听器类方法时，Laravel 将把这些方法注册为方法签名中类型提示的事件的事件监听器：

```php
use App\Events\PodcastProcessed;

class SendPodcastProcessedNotification
{
    /**
     * Handle the given event.
     *
     * @param  \App\Events\PodcastProcessed
     * @return void
     */
    public function handle(PodcastProcessed $event)
    {
        //
    }
}
```

默认情况下事件发现是禁用的，但是你可以通过覆盖你的应用程序 `EventServiceProvider` 类的 `shouldDiscoverEvents`方法来开启它：

```php
/**
 * 确定事件和监听器是否应当被自动发现。
 *
 * @return bool
 */
public function shouldDiscoverEvents()
{
    return true;
}
```

默认情况下，应用程序的监听器目录中的所有监听器都将被扫描。如果希望定义要扫描的其他目录，可以覆盖 `EventServiceProvider` 类中的 `discoverEventsWithin` 方法：

```php
/**
 * 获取应该用于发现事件的监听器目录。
 *
 * @return array
 */
protected function discoverEventsWithin()
{
    return [
        $this->app->path('Listeners'),
    ];
}
```

在生产中，你可能不希望框架在每次请求时扫描所有监听器。因此，在部署过程中，应该运行 `event:cache` Artisan 命令来缓存应用程序所有事件和监听器的清单。此清单将被通过框架使用以加快事件注册过程。`event:clear` 命令可用于销毁缓存。

{% hint style="info" %}

`event:list` 命令可以用于显示通过你的应用程序注册的所有事件和监听器的一个列表。

{% endhint %}

## 定义事件

事件类是一个数据容器，其中包含与事件相关的信息。例如，让我们假设生成的 `OrderShipped` 事件接收到一个 [Eloquent ORM](https://laravel.com/docs/5.8/eloquent) 对象：

```php
<?php

namespace App\Events;

use App\Order;
use Illuminate\Queue\SerializesModels;

class OrderShipped
{
    use SerializesModels;

    public $order;

    /**
     * 创建一个新事件实例。
     *
     * @param  \App\Order  $order
     * @return void
     */
    public function __construct(Order $order)
    {
        $this->order = $order;
    }
}
```

如你所见，这个事件类不包含逻辑。它是已购买 `Order` 实例的容器。如果使用 PHP 的 `serialize` 函数序列化事件对象，事件使用的 `SerializesModels` 特性将优雅地序列化任何 Eloquent 模型。

## 定义监听器

接下来，让我们看看示例事件的监听器。事件监听器在 `handle` 方法中接收事件实例。`event:generate` 命令将自动导入适当的事件类，并在 `handle` 方法上键入提示事件。在 `handle` 方法中，你可以执行响应事件所需的任何动作：

```php
<?php

namespace App\Listeners;

use App\Events\OrderShipped;

class SendShipmentNotification
{
    /**
     * 创建事件监听器。
     *
     * @return void
     */
    public function __construct()
    {
        //
    }

    /**
     * 处理事件。
     *
     * @param  \App\Events\OrderShipped  $event
     * @return void
     */
    public function handle(OrderShipped $event)
    {
        // 使用 $event->order 访问订单...
    }
}
```

{% hint style="info" %}

你的事件监听器还可以键入它们需要的对构造函数的依赖项。所有事件监听器都通过 Laravel 的 [服务容器](https://laravel.com/docs/5.8/container) 解析，因此依赖项将自动注入。

{% endhint %}

**停止事件的传播**

有时，你可能希望停止向其他监听器传播事件。你可以通过从监听器的 `handle` 方法返回 `false` 来这样做。

## 事件监听器队列

如果你的监听器要执行缓慢的任务，例如发送一个电子邮件或发出 HTTP 请求，则对监听器进行排队是有益的。在开始使用队列监听器之前，请确保 [配置你的队列](https://laravel.com/docs/5.8/queues)，并在服务器或本地开发环境上启动队列监听器。

要指定监听器应该排队，添加 `ShouldQueue` 接口到监听器类。通过 `event:generate` Artisan命令生成的监听器已经将此接口导入到当前名称空间中，因此你可以立即使用它：

```php
<?php

namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldQueue;

class SendShipmentNotification implements ShouldQueue
{
    //
}
```

就是这样！现在，当为某个事件调用此监听器时，事件调度程序将使用 Laravel 的 [队列系统](https://laravel.com/docs/5.8/queues) 自动对其进行排队。如果通过队列执行的监听器时没有抛出异常，则队列作业将在完成处理后自动删除。

**自定义队列连接 & 队列名称**

如果你希望自定义事件监听器的队列连接、队列名称或队列延迟时间，你可以在监听器类上定义 `$connection`、`$queue` 或 `$delay` 属性：

```php
<?php

namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldQueue;

class SendShipmentNotification implements ShouldQueue
{
    /**
     * 作业应发送到的连接的名称。
     *
     * @var string|null
     */
    public $connection = 'sqs';

    /**
     * 作业应发送到的队列的名称。
     *
     * @var string|null
     */
    public $queue = 'listeners';

    /**
     * 作业应当处理的延迟时间（秒）。
     *
     * @var int
     */
    public $delay = 60;
}
```

### 手动访问列队

如果需要手动访问监听器的底层队列作业的 `delete` 和 `release` 方法，可以使用 `Illuminate\Queue\InteractsWithQueue` 特性来这样做。默认情况下，在生成的监听器上导入此特性，并提供对这些方法的访问：

```php
<?php

namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Contracts\Queue\ShouldQueue;

class SendShipmentNotification implements ShouldQueue
{
    use InteractsWithQueue;

    /**
     * 处理事件。
     *
     * @param  \App\Events\OrderShipped  $event
     * @return void
     */
    public function handle(OrderShipped $event)
    {
        if (true) {
            $this->release(30);
        }
    }
}
```

### 处理失败的作业

有时你的排队的事件监听器可能会失败。如果队列监听器超过你的队列工作程序所定义的最大尝试次数，则将在你的监听器上调用 `failed` 的方法。`failed` 方法接收事件实例和导致失败的异常：

```php
<?php

namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Contracts\Queue\ShouldQueue;

class SendShipmentNotification implements ShouldQueue
{
    use InteractsWithQueue;

    /**
     * 处理事件。
     *
     * @param  \App\Events\OrderShipped  $event
     * @return void
     */
    public function handle(OrderShipped $event)
    {
        //
    }

    /**
     * 处理作业失败。
     *
     * @param  \App\Events\OrderShipped  $event
     * @param  \Exception  $exception
     * @return void
     */
    public function failed(OrderShipped $event, $exception)
    {
        //
    }
}
```

## 调度事件

要调度事件，你可以将事件的实例传递给 `event` 助手。助手将把事件调度给它的所有注册的监听器。由于 `event` 助手是全局可用的，你可以从应用程序中的任何位置调用它：

```php
<?php

namespace App\Http\Controllers;

use App\Order;
use App\Events\OrderShipped;
use App\Http\Controllers\Controller;

class OrderController extends Controller
{
    /**
     * 按订单发货。
     *
     * @param  int  $orderId
     * @return Response
     */
    public function ship($orderId)
    {
        $order = Order::findOrFail($orderId);

        // 订购出货逻辑...

        event(new OrderShipped($order));
    }
}
```

{% hint style="info" %}

在测试时，断言某些事件在没有实际触发其监听器的情况下被调度是有帮助的。Laravel 的 [内置测试助手](https://laravel.com/docs/5.8/mocking#event-fake) 使它变得很容易。

{% endhint %}

## 事件订阅
