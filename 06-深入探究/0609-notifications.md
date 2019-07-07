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

#### 其他通知的格式选项

你可以使用 `view` 方法指定一个自定义模板，而不是在通知类中定义文本的『行』，该模板应当用于渲染通知的电子邮件：

```php
/**
 * 获取通知的邮件表示形式。
 *
 * @param  mixed  $notifiable
 * @return \Illuminate\Notifications\Messages\MailMessage
 */
public function toMail($notifiable)
{
    return (new MailMessage)->view(
        'emails.name', ['invoice' => $this->invoice]
    );
}
```

此外，你可以从 `toMail` 方法返回一个 [可邮件对象](https://laravel.com/docs/5.8/mail)：

```php
use App\Mail\InvoicePaid as Mailable;

/**
 * 获取通知的邮件表示形式。
 *
 * @param  mixed  $notifiable
 * @return Mailable
 */
public function toMail($notifiable)
{
    return (new Mailable($this->invoice))->to($this->user->email);
}
```

#### 错误消息

有些通知会通知用户错误，比如发票付款失败。你可以通过在构建你的消息时调用 `error` 方法来指示邮件消息与错误有关。当对邮件消息使用 `error` 方法时，对操作的调用按钮将是红色而不是蓝色：

```php
/**
 * 获取通知的邮件表示形式。
 *
 * @param  mixed  $notifiable
 * @return \Illuminate\Notifications\Message
 */
public function toMail($notifiable)
{
    return (new MailMessage)
                ->error()
                ->subject('Notification Subject')
                ->line('...');
}
```

### 自定义发送者

默认情况下，电子邮件的发件人 / 发件人地址在 `config/mail.php` 配置文件中定义。但是，你可以使用 `from` 方法指定特定通知的发件人地址：

```php
/**
 * 获取通知的邮件表示形式。
 *
 * @param  mixed  $notifiable
 * @return \Illuminate\Notifications\Messages\MailMessage
 */
public function toMail($notifiable)
{
    return (new MailMessage)
                ->from('test@example.com', 'Example')
                ->line('...');
}
```

### 自定义接收者

当通过 `mail` 通道发送通知时，通知系统将自动查找你的应通知实体上的 `email` 属性。你可以通过在实体上定义一个 `routeNotificationForMail` 方法自定义用于投递通知的电子邮件地址：

```php
<?php

namespace App;

use Illuminate\Notifications\Notifiable;
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable
{
    use Notifiable;

    /**
     * 路由邮件通道的通知。
     *
     * @param  \Illuminate\Notifications\Notification  $notification
     * @return string
     */
    public function routeNotificationForMail($notification)
    {
        return $this->email_address;
    }
}
```

### 自定义主题

默认情况下，电子邮件的主题是格式化为『标题大写』的通知类名。因此，如果你的通知类名为 `InvoicePaid`，电子邮件的主题将是 `Invoice Paid`。如果希望为消息指定一个显式主题，可以在构建消息时调用 `subject` 方法：

```php
/**
 * 获取通知的邮件表示形式。
 *
 * @param  mixed  $notifiable
 * @return \Illuminate\Notifications\Messages\MailMessage
 */
public function toMail($notifiable)
{
    return (new MailMessage)
                ->subject('Notification Subject')
                ->line('...');
}
```

### 自定义模板

你可以通过发布通知包的资源来修改邮件通知使用的 HTML 和纯文本模板。运行此命令后，邮件通知模板将位于 `resources/views/vendor/notifications` 目录中：

```php
php artisan vendor:publish --tag=laravel-notifications
```

### 预览邮件通知

在设计邮件通知模板时，可以方便地像典型的 Blade 模板一样在浏览器中快速预览渲染的邮件消息。因此，Laravel 允许你直接从路由闭包或控制器返回由邮件通知生成的任何邮件消息。当返回 `MailMessage` 时，它将在浏览器中渲染和显示，允许你快速预览其设计，而不需要将其发送到实际的电子邮件地址：

```php
Route::get('mail', function () {
    $invoice = App\Invoice::find(1);

    return (new App\Notifications\InvoicePaid($invoice))
                ->toMail($invoice->user);
});
```

## Markdown 邮件通知

Markdown 邮件通知允许你利用预先构建的邮件通知模板，同时允许你更自由地编写更长的自定义消息。由于消息是用 Markdown 编写的，所以 Laravel 能够为消息呈现漂亮的、响应性强的 HTML 模板，同时还能自动生成纯文本副本。

### 生成消息

要生成具有相应 Markdown 模板的通知，可以使用 `make:notification` Artisan 命令的 `——markdown` 选项：

```bash
php artisan make:notification InvoicePaid --markdown=mail.invoice.paid
```

与所有其他邮件通知一样，使用 Markdown 模板的通知应该在其通知类上定义一个 `toMail` 方法。但是，不要使用 `line` 和 `action` 方法来构造通知，而是使用 `markdown` 方法来指定应该使用的 Markdown 模板的名称：

```php
/**
 * 获取通知的邮件表现形式。
 *
 * @param  mixed  $notifiable
 * @return \Illuminate\Notifications\Messages\MailMessage
 */
public function toMail($notifiable)
{
    $url = url('/invoice/'.$this->invoice->id);

    return (new MailMessage)
                ->subject('Invoice Paid')
                ->markdown('mail.invoice.paid', ['url' => $url]);
}
```

### 编写消息

Markdown 邮件通知使用 Blade 组件和 Markdown 语法的组合，允许你在利用 Laravel 预先设计的通知组件的同时轻松构造通知：

```php
@component('mail::message')
# Invoice Paid

Your invoice has been paid!

@component('mail::button', ['url' => $url])
View Invoice
@endcomponent

Thanks,<br>
{{ config('app.name') }}
@endcomponent
```

#### 按钮组件

按钮组件渲染一个居中按钮链接。组件接受两个参数，一个 `url` 和一个可选 `color`。支持的颜色是 `blue`、`green` 和 `red`。你可以向通知添加任意数量的按钮组件：

```html
@component('mail::button', ['url' => $url, 'color' => 'green'])
View Invoice
@endcomponent
```

#### 面板组件

面板组件在一个背景颜色与通知的其余部分略有不同的面板中渲染给定的文本块。这允许你将注意力吸引到给定的文本块上：

```html
@component('mail::panel')
This is the panel content.
@endcomponent
```

#### 表格组件

表格组件允许你将 Markdown 表转换为 HTML 表格。组件接受 Markdown 表格作为其内容。使用默认的 Markdown 表对齐语法支持表格列对齐：

```html
@component('mail::table')
| Laravel  | Table         | Example |
| -------- | ------------- | ------- |
| Col 2 is | Centered      | $10     |
| Col 3 is | Right-Aligned | $20     |
@endcomponent
```

### 自定义组件

你可以将所有 Markdown 通知组件导出到你自己的应用程序进行定制。要导出组件，使用 `vendor:publish` Artisan 命令发布 `laravel-mail` 资产标记：

```bash
php artisan vendor:publish --tag=laravel-mail
```

此命令将 Markdown 邮件组件发布到 `resources/views/vendor/mail` 目录。`mail` 目录将包含一个 `html` 和一个 `text` 目录，每个目录包含它们各自可用组件的相应表示。你可以随意自定义这些组件。

#### 自定义 CSS

导出组件后，`resources/views/vendor/mail/html/themes` 目录将包含 `default.css` 文件。你可以在此文件中自定义 CSS，并且你的样式将自动在 Markdown 通知的 HTML 表示中内联。

{% hint style="info" %}

如果你想为 Markdown 组件构建一个全新的主题，请在 `html/themes` 目录中编写一个新的 CSS 文件，并更改 `mail` 配置文件的 `theme` 选项。

{% endhint %}

## 数据库通知

### 先决条件

`database` 通知通道将通知信息存储在数据库表中。该表将包含通知类型等信息以及描述通知的自定义 JSON 数据。

你可以查询该表以在应用程序的用户界面中显示通知。但是，在此之前，你需要创建一个数据库表来保存你的通知。你可以使用 `notification:table` 命令生成具有合适表模式的迁移：

```bash
php artisan notifications:table

php artisan migrate
```

### 格式化数据库通知

如果通知支持存储在数据库表中，则应在通知类上定义 `toDatabase` 或 `toArray` 方法。该方法将接收一个 `$notifiable` 实体，并返回一个普通的 PHP 数组。返回的数组将被编码为 JSON 并存储在 `notifications` 表的 `data` 列中。让我们看一个 `toArray` 方法的例子：

```php
/**
 * 获取通知的数组表示形式。
 *
 * @param  mixed  $notifiable
 * @return array
 */
public function toArray($notifiable)
{
    return [
        'invoice_id' => $this->invoice->id,
        'amount' => $this->invoice->amount,
    ];
}
```

#### `toDatabase` VS `toArray`

`broadcast` 通道还使用 `toArray` 方法来确定要向 JavaScript 客户端广播哪些数据。如果希望 `database` 和 `broadcast` 频道有两种不同的数组表示，应该定义 `toDatabase` 方法，而不是 `toArray` 方法。

### 访问通知

一旦通知存储在数据库中，你就需要一种方便的方法来从你的通知实体上访问它们。在 Laravel 的默认 `App\User` 模型中包含了一个 `Illuminate\Notifications\Notifiable` 特性，它包含一个 `notifications` Eloquent 关系，可以为实体返回通知。要获取通知，你可以像访问其他任何 Eloquent 关系一样访问此方法。默认情况下，通知将根据 `created_at` 的时间戳进行排序：

```php
$user = App\User::find(1);

foreach ($user->notifications as $notification) {
    echo $notification->type;
}
```

如果只想检索『未读』通知，你可以使用 `unreadNotifications` 关系。同样，这些通知将根据 `created_at` 的时间戳进行排序：

```php
$user = App\User::find(1);

foreach ($user->unreadNotifications as $notification) {
    echo $notification->type;
}
```

{% hint style="info" %}

要从 JavaScript 客户端访问通知，你应该为应用程序定义一个通知控制器，该控制器返回可通知实体（如当前用户）的通知。然后，你可以从 JavaScript 客户端向该控制器的 URI 发出一个 HTTP 请求。

{% endhint %}

### 通知标记为已读

通常，你希望在用户查看通知时将其标记为『已读』。`Illuminate\Notifications\Notifiable` 特性提供了一个 `markAsRead` 方法，该方法更新通知数据库记录上的 `read_at` 列：

```php
$user = App\User::find(1);

foreach ($user->unreadNotifications as $notification) {
    $notification->markAsRead();
}
```

但是，你可以直接在通知集合上使用 `markAsRead` 方法，而不是遍历每个通知：

```php
$user->unreadNotifications->markAsRead();
```

你还可以使用大量更新将查询的所有通知标记为已读，而不从数据库中检索它们：

```php
$user = App\User::find(1);

$user->unreadNotifications()->update(['read_at' => now()]);
```

你可以 `delete` 通知以将它们从表中完全删除：

```php
$user->notifications()->delete();
```

## 广播通知

### 先决条件

在广播通知之前，你应该配置并熟悉 Laravel 的 [事件广播](https://laravel.com/docs/5.8/broadcasting) 服务。事件广播提供了一种方法来响应来自 JavaScript 客户端的服务器端触发的 Laravel 事件。

## SMS 通知

## Slack 通知

## 本地化通知

## 通知事件

## 自定义通道
