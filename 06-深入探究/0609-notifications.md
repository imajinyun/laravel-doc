# 通知

## 简介

除了支持 [发送电子邮件](https://laravel.com/docs/5.8/mail) 之外，Laravel 还支持通过多种投递渠道发送通知，包括邮件、SMS（通过 [Nexmo](https://www.nexmo.com/)）和 [Slack](https://slack.com/intl/en-cn/)。通知也可以存储在数据库中，因此它们可以显示在 Web 界面中。

通常，通知应该是简短的信息消息，通知用户应用程序中发生的一些事情。例如，如果你正在编写一个计费应用程序，您可以通过电子邮件和 SMS 通道向用户发送一个『已付款发票』通知。

## 创建通知

在 Laravel 中，每个通知都由一个类表示（通常存储在 `app/Notifications` 目录中）。如果你在应用程序中没有看到此目录，请不要担心，当你运行 `make:notification` Artisan 命令时，将为你创建该目录：

```bash
php artisan make:notification InvoicePaid
```

此命令将在你的 `app/Notifications` 目录中放置一个新的通知类。每个通知类都包含一个 `via` 方法和一个可变数量的消息构建方法（例如：`toMail` 或 `toDatabase`），这些方法将通知转换为针对该特定通道优化的消息。

## 发送通知

### 使用通知特性

通知可以通过两种方式发送：使用 `Notifiable` 特性的 `notify` 方法或使用 `Notification` [外观](https://laravel.com/docs/5.8/facades)。首先，让我们探索一下使用这个特性：

```php
<?php

namespace App;

use Illuminate\Notifications\Notifiable;
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable
{
    use Notifiable;
}
```

默认的 `App\User` 模型使用这个特性，它包含一个可以用来发送通知的方法：`notify`。`notify` 方法期望接收一个通知实例：

```php
use App\Notifications\InvoicePaid;

$user->notify(new InvoicePaid($invoice));
```

{% hint style="info" %}

记住，你可以在任何你的模型上使用 `Illuminate\Notifications\Notifiable` 特性。你不要被限制于将它包含在你的 `User` 模型中。

{% endhint %}

### 使用通知外观

或者，你可以通过 `Notification` [外观](https://laravel.com/docs/5.8/facades) 发送通知。当你需要向多个应通知实体（比如：一个用户集合）发送通知时，这是非常有用的。要使用外观发送通知，将所有应通知的实体和通知实例传递给 `send` 方法：

```php
Notification::send($users, new InvoicePaid($invoice));
```

### 指定投递通道

每个通知类都有一个 `via` 方法，用于确定通知将在哪些通道上传递。可以在 `mail`、`database`、`broadcast`、`nexmo` 和 `slack` 通道上发送通知。

{% hint style="info" %}

如果你想使用其他投递渠道，如：Telegram 或 Pusher，请查看社区驱动的 [Laravel 通知通道网站](http://laravel-notification-channels.com/)。

{% endhint %}

`via` 方法接收一个 `$notifiable` 实例，该实例将是正在向其发送通知的类的实例。你可以使用 `$notifiable` 来确定应该在哪个通道上投递通知：

```php
/**
 * 获取通知的投递通道。
 *
 * @param  mixed  $notifiable
 * @return array
 */
public function via($notifiable)
{
    return $notifiable->prefers_sms ? ['nexmo'] : ['mail', 'database'];
}
```

### 排队通知

{% hint style="danger" %}

在对通知进行排队之前，你应该配置队列并 [开启一个工作者](https://laravel.com/docs/5.8/queues)。

{% endhint %}

发送通知可能会花费时间，特别是当通道需要外部 API 调用来传递通知时。为了加快应用程序的响应时间，可以通过将 `ShouldQueue` 接口和 `Queueable` 特性添加到类中来让通知排队。对于使用 `make:notification` 生成的所有通知，已经导入了接口和特性，因此你可以立即将它们添加到你的通知类中：

```php
<?php

namespace App\Notifications;

use Illuminate\Bus\Queueable;
use Illuminate\Notifications\Notification;
use Illuminate\Contracts\Queue\ShouldQueue;

class InvoicePaid extends Notification implements ShouldQueue
{
    use Queueable;

    // ...
}
```

一旦将 `ShouldQueue` 接口添加到你的通知中，就可以像平常一样发送通知。Laravel 将检测类上的 `ShouldQueue` 接口，并自动对通知的投递进行排队：

```php
$user->notify(new InvoicePaid($invoice));
```

如果希望延迟通知的投递，可以将 `delay` 方法链接到你的通知实例上：

```php
$when = now()->addMinutes(10);

$user->notify((new InvoicePaid($invoice))->delay($when));
```

### 按需通知

有时，你可能需要向没有存储为应用程序『用户』的人发送通知。使用 `Notification::route` 方法，你可以在发送通知之前指定特定的通知路由信息：

```php
Notification::route('mail', 'taylor@example.com')
            ->route('nexmo', '5555555555')
            ->notify(new InvoicePaid($invoice));
```

## 邮件通知

### 格式化邮件消息

如果通知支持以电子邮件的形式发送，则应该在通类上定义一个 `toMail` 方法。该方法将接收一个 `$notifiable` 实体，并返回一个 `Illuminate\Notifications\Messages\MailMessage` 实例。邮件消息可能包含几行文本以及一个『动作调用』。让我们看一个 `toMail` 方法的例子：

```php
/**
 * 获取通知的邮件表示形式。
 *
 * @param  mixed  $notifiable
 * @return \Illuminate\Notifications\Messages\MailMessage
 */
public function toMail($notifiable)
{
    $url = url('/invoice/'.$this->invoice->id);

    return (new MailMessage)
                ->greeting('Hello!')
                ->line('One of your invoices has been paid!')
                ->action('View Invoice', $url)
                ->line('Thank you for using our application!');
}
```

{% hint style="info" %}

注意，我们在 `toMail` 方法中使用 `$this->invoice->id`。你可以将你的通知需要去生成其消息的任何数据传递到通知的构造函数中。

{% endhint %}

在本例中，我们注册了一个问候语、一行文本、一个动作调用，然后是另一行文本。通过 `MailMessage` 对象提供的这些方法使格式化小型事务性电子邮件变得简单和快速。然后，邮件通道将消息组件转换为具有纯文本对应的响应良好的 HTML 电子邮件模板。下面是由 `mail` 通道生成的电子邮件的示例：

![notification-example.png](https://entities.oss-cn-beijing.aliyuncs.com/laravel/docs/5.8/notification-example.png)

{% hint style="info" %}

发送邮件通知时，请务必在 `config/app.php` 配置文件中设置 `name` 值。此值将用于邮件通知消息的页眉和页脚。

{% endhint %}

### 自定义发送者

### 自定义接收者

### 自定义主题

### 自定义模板

### 预览邮件通知

## Markdown 邮件通知

## 数据库通知

## 广播通知

## SMS 通知

## Slack 通知

## 本地化通知

## 通知事件

## 自定义通道
