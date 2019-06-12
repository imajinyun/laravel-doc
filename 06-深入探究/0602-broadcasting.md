# 广播

## 简介

在许多现代 Web 应用程序中，WebSockets 用于实现实时的、实时更新的用户界面。当在服务器上更新一些数据时，消息通常通过 WebSocket 连接发送并由客户机处理。这为不断轮询应用程序的更改提供了更健壮、更有效的替代方法。

为了帮助我构建这些类型的应用程序，Laravel 使通过 WebSocket 连接『广播』你的 [事件](https://laravel.com/docs/5.8/events) 变得很容易。广播你的 Laravel 事件允许你在服务器端代码和客户端 JavaScript 应用程序之间共享相同的事件名称。

{% hint style="info" %}

在开始事件广播之前，确保你已经阅读了有关 Laravel [事件和侦听器](https://laravel.com/docs/5.8/events) 的所有文档。

{% endhint %}

### 配置

你的所有应用程序的事件广播配置都存储在 `config/broadcasting.php` 配置文件中。Laravel 支持多种开箱即用的广播驱动程序：[Pusher](https://pusher.com/)，[Redis](https://laravel.com/docs/5.8/redis) 和用于本地开发和调试的 `log` 驱动程序。此外，还包括一个 `null` 驱动程序，允许你完全禁用广播。`config/broadcasting.php` 配置文件中的每个驱动程序都包含一个配置示例。

#### 广播服务提供者

在广播任何事件之前，你首先需要注册 `App\Providers\BroadcastServiceProvider`。在新的 Laravel 应用程序中，你只需要在 `config/app.php` 配置文件的 `providers` 数组中取消注释此提供程序。此提供程序将允许你注册广播授权路由和回调。

#### CSRF 令牌

[Laravel Echo](https://laravel.com/docs/5.8/broadcasting#installing-laravel-echo) 将需要访问当前会话的 CSRF 令牌。你应该验证你的应用程序的 `head` HTML 元素定义一个包含 CSRF 令牌的 `meta` 标记：

```html
<meta name="csrf-token" content="{{ csrf_token() }}">
```

### 驱动先决条件

#### Pusher

如果要经过 [Pusher](https://pusher.com/) 广播你的事件，应该使用 Composer 包管理器安装 Pusher PHP SDK：

```bash
composer require pusher/pusher-php-server "~3.0"
```

接下来，你应该在 `config/broadcasting.php` 配置文件中配置 Pusher 凭据。此文件中已包含 Pusher 配置示例，允许你快速指定 Pusher 键，密钥和应用程序 ID。`config/broadcast.php` 文件的 `pusher` 配置还允许你指定 Pusher 支持的其他 `options`，例如群集：

```php
'options' => [
    'cluster' => 'eu',
    'encrypted' => true
],
```

当你使用 Pusher 和 [Laravel Echo](https://laravel.com/docs/5.8/broadcasting#installing-laravel-echo)，你应当在你的 `resources/js/bootstrap.js` 文件中实例化 Echo 实例并指定 `pusher` 作为你期望的广播者：

```js
import Echo from "laravel-echo";

window.Pusher = require('pusher-js');

window.Echo = new Echo({
    broadcaster: 'pusher',
    key: 'your-pusher-key'
});
```

#### Redis

如果你使用 Redis 作为广播者，你应当安装 Predis 库：

```bash
composer require predis/predis
```

Redis 广播者将使用 Redis 的发布 / 订阅功能广播消息；但是，你需要将其与 WebSocket 服务器配对，该服务器可以从 Redis 接收消息并将它们广播到你的 WebSocket 频道。

当 Redis 广播者发布事件时，它将发布在事件的指定通道名称上，有效负载将是 JSON 编码的字符串，包含事件名称、`data` 负载和生成事件套接字 ID 的用户（如果适用）。

#### Socket.IO

如果你要将 Redis 广播者与一个 Socket.IO 服务器配对，您将需要在你的应用程序中包含 Socket.IO JavaScript 客户端库。你可以通过 NPM 包管理器安装它：

```bash
npm install --save socket.io-client
```

接下来，你将需要用 `socket.io` 去实例化 Echo 连接器和 `host`。

```js
import Echo from "laravel-echo"

window.io = require('socket.io-client');

window.Echo = new Echo({
    broadcaster: 'socket.io',
    host: window.location.hostname + ':6001'
});
```

最后，你需要运行兼容的 Socket.IO 服务器。Laravel 不包含 Socket.IO 服务器实现；但是，社区驱动的 Socket.IO 服务器目前在 [tlaverdure/laravel-echo-server](https://github.com/tlaverdure/laravel-echo-server) GitHub 仓库中维护。

#### 队列先决条件

在广播事件之前，还需要配置和运行 [队列侦听器](https://laravel.com/docs/5.8/queues)。所有事件广播都是通过队列作业完成的，因此应用程序的响应时间不会受到严重影响。

## 概念概述

Laravel 的事件广播允许你使用基于驱动程序的 WebSockets 方法将服务器端 Laravel 事件广播到客户端 JavaScript 应用程序。目前，Laravel 与 [Pusher](https://pusher.com/) 和 Redis 驱动器一起发货。可以使用 [Laravel Echo](https://laravel.com/docs/5.8/broadcasting#installing-laravel-echo) Javascript 包在客户端轻松使用这些事件。

事件通过『通道』广播，可以指定为公共的或私有的。任何你的应用程序的访问者，可以在没有任何认证或授权的情况下订阅公共频道；但是，为了订阅一个私有通道，必须对用户进行认证并授权他在该通道上监听。

### 使用一个示例应用程序

在深入探究事件广播的每个组件之前，让我们以电子商务商店为例进行一个高层次的概述。我们不会讨论配置 [Pusher](https://pusher.com/) 或 [Laravel Echo](https://laravel.com/docs/5.8/broadcasting#installing-laravel-echo) 的细节，因为这将在本文档的其他部分中详细讨论。

在我们的应用程序中，让我们假设有一个页面允许用户查看订单的发货状态。我们还假设，当应用程序处理发货状态更新时，一个 `ShippingStatusUpdated` 事件被触发:

```php
event(new ShippingStatusUpdated($update));
```

#### `ShouldBroadcast` 接口

当用户查看他们的订单时，我们不希望他们必须刷新页面才能查看状态更新。相反，我们希望在应用程序创建时向其广播更新。因此，我们需要用 `ShouldBroadcast` 接口标记 `ShippingStatusUpdated` 事件。这将指示 Laravel 在事件触发时广播该事件：

```php
<?php

namespace App\Events;

use Illuminate\Broadcasting\Channel;
use Illuminate\Queue\SerializesModels;
use Illuminate\Broadcasting\PrivateChannel;
use Illuminate\Broadcasting\PresenceChannel;
use Illuminate\Broadcasting\InteractsWithSockets;
use Illuminate\Contracts\Broadcasting\ShouldBroadcast;

class ShippingStatusUpdated implements ShouldBroadcast
{
    /**
     * 关于发货状态更新的信息。
     *
     * @var string
     */
    public $update;
}
```

`ShouldBroadcast` 接口要求我们的事件定义成一个 `broadcastOn` 方法。此方法负责返回事件应该广播的通道。这个方法的空存根已经在生成的事件类上定义，所以我们只需要填充它的细节。我们只希望订单的创建者能够查看状态更新，因此我们将在与订单绑定的私有通道上广播事件：

```php
/**
 * 获取事件应该广播的通道。
 *
 * @return \Illuminate\Broadcasting\PrivateChannel
 */
public function broadcastOn()
{
    return new PrivateChannel('order.'.$this->update->order_id);
}
```

#### 授权通道

请记住，用户必须有权通过私人通道收听。我们可以在 `routes/channels.php` 文件中定义我们的通道授权规则。在此示例中，我们需要验证尝试侦听私有 `order.1` 频道的任何用户实际上是订单的创建者：

```php
Broadcast::channel('order.{orderId}', function ($user, $orderId) {
    return $user->id === Order::findOrNew($orderId)->user_id;
});
```

`channel` 方法接受两个参数：通道的名称和一个回调函数，回调函数返回 `true` 或 `false`，指示是否授权用户监听通道。

所有授权回调都将当前经过认证的用户作为它们的第一个参数，任何其他通配符参数作为它们的后续参数。在本例中，我们使用 `{orderId}` 占位符来指示通道名称的『ID』部分是通配符。

#### 监听事件广播

接下来，剩下的就是侦听 JavaScript 应用程序中的事件。我们可以使用 Laravel Echo 来实现这一点。首先，我们将使用 `private` 方法订阅私有通道。然后，我们可以使用 `listen` 方法侦听 `ShippingStatusUpdated` 事件。默认情况下，事件的所有公共属性都将包含在广播事件中：

```js
Echo.private(`order.${orderId}`)
    .listen('ShippingStatusUpdated', (e) => {
        console.log(e.update);
    });
```

## 定义广播事件

要通知 Laravel 某个给定的事件应该被广播，要在事件类上实现 `Illuminate\Contracts\Broadcasting\ShouldBroadcast` 接口。这个接口已经导入到通过框架生成的所有事件类中，因此你可以轻松地将它添加到任何你的事件中。

`ShouldBroadcast` 接口要求你实现一个方法：`broadcastOn`。`broadcastOn` 方法应该返回事件应该广播的通道或通道数组。通道应该是 `Channel`，`PrivateChannel` 或 `PresenceChannel` 的实例。`Channel` 实例表示任何用户都可以订阅的公共通道，而 `PrivateChannels` 和 `PresenceChannels` 表示需要 [通道授权](https://laravel.com/docs/5.8/broadcasting#authorizing-channels) 的私有通道：

```php
<?php

namespace App\Events;

use App\User;
use Illuminate\Broadcasting\Channel;
use Illuminate\Queue\SerializesModels;
use Illuminate\Broadcasting\PrivateChannel;
use Illuminate\Broadcasting\PresenceChannel;
use Illuminate\Broadcasting\InteractsWithSockets;
use Illuminate\Contracts\Broadcasting\ShouldBroadcast;

class ServerCreated implements ShouldBroadcast
{
    use SerializesModels;

    public $user;

    /**
     * 创建一个新的事件实例。
     *
     * @return void
     */
    public function __construct(User $user)
    {
        $this->user = $user;
    }

    /**
     * 获取事件应该广播的通道。
     *
     * @return Channel|array
     */
    public function broadcastOn()
    {
        return new PrivateChannel('user.'.$this->user->id);
    }
}
```

然后，你只需要像往常一样 [触发事件](https://laravel.com/docs/5.8/events)。一旦事件被触发，[队列作业](https://laravel.com/docs/5.8/queues) 将自动通过指定的广播驱动程序广播事件。

### 广播名称

默认情况下，Laravel 将使用事件的类名广播事件。但是，你可以通过在事件上定义一个 `broadcastAs` 方法来定制广播名称：

```php
/**
 * 事件的广播名称。
 *
 * @return string
 */
public function broadcastAs()
{
    return 'server.created';
}
```

如果使用 `broadcastAs` 方法自定义广播名称，你应当确保用一个前导 `.` 符号注册你的侦听器。这将指示 Echo 不要将应用程序的命名空间预先添加到事件中：

```js
.listen('.server.created', function (e) {
    ....
});
```

### 广播数据

当一个事件被广播时，它的所有 `public` 属性都会自动序列化并作为事件的有效负载进行广播，允许你从你的 JavaScript 应用程序访问它的任何公共数据。因此，例如，如果你的事件有一个包含 Eloquent 模型的公共 `$user` 属性，则事件的广播有效负载将是：

```json
{
    "user": {
        "id": 1,
        "name": "Patrick Stewart"
        ...
    }
}
```

但是，如果希望对广播负载有更细粒度的控制，可以向事件添加 `broadcastWith` 方法。此方法应该返回希望作为事件有效负载广播的数据数组：

```php
/**
 * 获取广播的数据。
 *
 * @return array
 */
public function broadcastWith()
{
    return ['id' => $this->user->id];
}
```

### 广播队列

### 广播条件

## 授权通道

## 广播事件

## 接收广播

## 在场通道

## 客户端事件

## 通知
