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

## 定义事件

## 定义监听器

## 事件监听器队列

## 调度事件

## 事件订阅
