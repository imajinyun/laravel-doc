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

默认情况下，每个广播事件都放置在默认队列上，用于默认的队列连接在你的 `queue.php` 配置文件中指定的。你可以通过在你的事件类上定义一个 `broadcastQueue` 属性来定制广播器使用的队列。此属性应指定你希望在广播时使用的队列的名称：

```php
/**
 * 放置事件的队列的名称。
 *
 * @var string
 */
public $broadcastQueue = 'your-queue-name';
```

如果你想使用 `sync` 队列广播你的事件而不是默认的队列驱动，你可以实现 `ShouldBroadcastNow` 接口而不是 `ShouldBroadcast`：

```php
<?php

use Illuminate\Contracts\Broadcasting\ShouldBroadcastNow;

class ShippingStatusUpdated implements ShouldBroadcastNow
{
    //
}
```

### 广播条件

有时，只有在给定条件为真时，才希望广播你的事件。你可以通过向你的事件类添加 `broadcastWhen` 方法来定义这些条件：

```php
/**
 * 决定该事件是否应当广播。
 *
 * @return bool
 */
public function broadcastWhen()
{
    return $this->value > 100;
}
```

## 授权通道

私有通道要求你授权当前经过认证的用户可以在通道上实际侦听。这是通过使用通道名称向 Laravel 应用程序发出 HTTP 请求，并允许应用程序确定用户是否可以在该通道上侦听来实现的。当使用 [Laravel Echo](https://laravel.com/docs/5.8/broadcasting#installing-laravel-echo) 时，将自动发出 HTTP 请求来授权对私有通道的订阅；但是，你确实需要定义正确的路由来响应这些请求。

### 定义认证路由

值得庆幸的是，Laravel 可以轻松定义响应通道授权请求的路由。在 Laravel 应用程序附带的 `BroadcastServiceProvider` 中，你将看到对 `Broadcast::routes` 方法的调用。此方法将注册 `/broadcast/auth` 路由以处理授权请求：

```php
Broadcast::routes();
```

`Broadcast::routes` 方法将自动将其路由放置在 `web` 中间件组中；但是，如果希望自定义分配的属性，可以将路由属性数组传递到该方法：

```php
Broadcast::routes($attributes);
```

#### 自定义授权端点

默认情况下，Echo 将使用 `/broadcast/auth` 端点来授权通道访问。但是，你可以通过将 `authEndpoint` 配置选项传递给 Echo 实例来指定自己的授权端点：

```js
window.Echo = new Echo({
    broadcaster: 'pusher',
    key: 'your-pusher-key',
    authEndpoint: '/custom/endpoint/auth'
});
```

### 定义认证回调

接下来，我们需要定义实际执行通道授权的逻辑。这是在你的应用程序附带的 `routes/channels.php` 文件中完成的。在此文件中，您可以使用 `Broadcast::channel` 方法注册通道授权回调：

```php
Broadcast::channel('order.{orderId}', function ($user, $orderId) {
    return $user->id === Order::findOrNew($orderId)->user_id;
});
```

`channel` 方法接受两个参数：通道的名称和一个回调函数，回调函数返回 `true` 或 `false`，指示是否授权用户在通道上去监听。

所有授权回调都将当前经过认证的用户作为它们的第一个参数，任何其他通配符参数作为它们的后续参数。在本例中，我们使用 `{orderId}` 占位符来指示通道名称的『ID』部分是通配符。

#### 授权回调模型绑定

就像 HTTP 路由一样，通道路由也可以利用隐式和显式 [路由模型绑定](https://laravel.com/docs/5.8/routing#route-model-binding)。例如：你可以请求一个实际的 `Order` 模型实例，而不是接收字符串或数字 order ID：

```php
use App\Order;

Broadcast::channel('order.{order}', function ($user, Order $order) {
    return $user->id === $order->user_id;
});
```

#### 授权回调认证

私有和存在广播通道通过应用程序的默认认证守卫对当前用户进行认证。如果用户没有经过认证，则会自动拒绝通道授权，并且永远不会执行授权回调。但是，如果必要，你可以分配多个自定义守卫来认证传入的请求：

```php
Broadcast::channel('channel', function() {
    // ...
}, ['guards' => ['web', 'admin']])
```

### 定义通道类

如果你的应用程序消耗了许多不同的通道，则你的 `routes/channels.php` 文件可能会变得笨重。因此，你可以使用通道类，而不是使用 Closures 来授权通道。要生成通道类，请使用 `make:channel` Artisan 命令。此命令将在 `App/Broadcasting` 目录中放置一个新的通道类。

```bash
php artisan make:channel OrderChannel
```

接下来，在你的 `routes/channels.php` 文件中注册你的通道：

```php
use App\Broadcasting\OrderChannel;

Broadcast::channel('order.{order}', OrderChannel::class);
```

最后，你可以将通道的授权逻辑放在通道类的 `join` 方法中。此 `join` 方法将包含通常放置在你的通道授权闭包中的相同逻辑。你还可以利用通道模型绑定：

```php
<?php

namespace App\Broadcasting;

use App\User;
use App\Order;

class OrderChannel
{
    /**
     * 创建一个新的通道实例。
     *
     * @return void
     */
    public function __construct()
    {
        //
    }

    /**
     * 认证用户对通道的访问权限。
     *
     * @param  \App\User  $user
     * @param  \App\Order  $order
     * @return array|bool
     */
    public function join(User $user, Order $order)
    {
        return $user->id === $order->user_id;
    }
}
```

{% hint style="info" %}

与 Laravel 中的许多其他类一样，通道类将由 [服务容器](https://laravel.com/docs/5.8/container) 自动解析。因此，你可以在通道的构造函数中键入提示通道所需的任何依赖项。

{% endhint %}

## 广播事件

一旦定义了一个事件并使用 `ShouldBroadcast` 接口对其进行了标记，就只需要使用 `event` 函数触发该事件。事件调度程序将注意到事件被标记为 `ShouldBroadcast` 接口，并将事件排队进行广播：

```php
event(new ShippingStatusUpdated($update));
```

### 仅给别人

在构建使用事件广播的应用程序时，可以使用 `broadcast` 函数取代 `event` 函数。与 `event` 函数一样，`broadcast` 函数将事件调度给服务器端侦听器：

```php
broadcast(new ShippingStatusUpdated($update));
```

不过，`broadcast` 函数还暴露了 `toOthers` 方法，该方法允许你将当前用户从广播的接收方中排除：

```php
broadcast(new ShippingStatusUpdated($update))->toOthers();
```

为了更好地了解何时可能需要使用 `toOthers` 方法，让我们设想一个任务列表应用程序，用户可以通过输入任务名称来创建新任务。要创建任务，你的应用程序可能会向 `/task` 端点发出请求，该端点广播任务的创建并返回新任务的 JSON 表示。当你的 JavaScript 应用程序从端点收到响应时，它可能会直接将新任务插入其任务列表中，如下所示：

```php
axios.post('/task', task)
    .then((response) => {
        this.tasks.push(response.data);
    });
```

但是，请记住，我们还广播了任务的创建。如果 JavaScript 应用程序正在侦听此事件，以便将任务添加到任务列表中，则列表中会有重复的任务：一个来自端点，一个来自广播。你可以使用 `toOthers` 方法来解决这个问题，该方法指示广播程序不要将事件广播给当前用户。

#### 配置

当你初始化 Laravel Echo 实例时，将套接字 ID 分配给连接。如果你正在使用 [Vue](https://vuejs.org/) 和 [Axios](https://github.com/axios/axios)，套接字 ID 将自动作为 `X-Socket-ID` 头附加到每个传出请求上。然后，当你调用 `toOthers` 方法时，Laravel 将从报头中提取套接字 ID，并指示广播器不要与该套接字 ID 的任何连接广播。

如果不使用 Vue 和 Axios，则需要手动配置 JavaScript 应用程序来发送 `X-Socket-ID` 头。你可以检索套接字 ID 使用 `Echo.socketId` 方法：

```js
var socketId = Echo.socketId();
```

## 接收广播

### 安装 Laravel Echo

Laravel Echo 是一个 JavaScript 库，可以轻松订阅频道并侦听 Laravel 广播的事件。你可以通过 NPM 包管理器安装 Echo。在本例中，我们还将安装 `pusher-js` 包，因为我们将使用 Pusher 广播器：

```bash
npm install --save laravel-echo pusher-js
```

安装 Echo 后，你就可以在应用程序的 JavaScript 中创建一个全新的 Echo 实例。这样做的好地方是 Laravel 框架附带的 `resources/js/bootstrap.js` 文件的底部：

```js
import Echo from "laravel-echo"

window.Echo = new Echo({
    broadcaster: 'pusher',
    key: 'your-pusher-key'
});
```

在创建使用 `pusher` 连接器的 Echo 实例时，你还可以指定一个 `cluster` 以及是否应该加密连接:

```js
window.Echo = new Echo({
    broadcaster: 'pusher',
    key: 'your-pusher-key',
    cluster: 'eu',
    encrypted: true
});
```

#### 使用存在的客户端实例

如果你已经有一个 Pusher 或 Socket.io 客户端实例，你希望 Echo 利用该实例，你可以通过 `client` 配置选项将其传递给 Echo：

```js
const client = require('pusher-js');

window.Echo = new Echo({
    broadcaster: 'pusher',
    key: 'your-pusher-key',
    client: client
});
```

### 监听事件

一旦你安装并实例化 Echo 之后，就可以开始监听事件广播了。首先，使用 `channel` 方法检索一个通道的实例，然后调用 `listen` 方法侦听指定的事件：

```js
Echo.channel('orders')
    .listen('OrderShipped', (e) => {
        console.log(e.order.name);
    });
```

如果希望侦听私有通道上的事件，要使用 `private` 方法替代。你可以继续对 `listen` 方法进行链式的调用，以侦听单个通道上的多个事件：

```php
Echo.private('orders')
    .listen(...)
    .listen(...)
    .listen(...);
```

### 离开一个通道

要离开一个通道，你需要在你的 Echo 实例上调用 `leaveChannel` 方法：

```js
Echo.leaveChannel('orders');
```

如果希望离开通道及其关联的私有通道和存在通道，可以调用 `leave` 方法：

```js
Echo.leave('orders');
```

### 命名空间

我可能已经注意到在上面的示例中，我们没有为事件类指定完整的名称空间。这是因为 Echo 将自动假定事件位于 `App\Events` 命名空间中。但是，你可以在通过传递 `namespace` 配置选项实例化 Echo 时，可以配置根命名空间：

```js
window.Echo = new Echo({
    broadcaster: 'pusher',
    key: 'your-pusher-key',
    namespace: 'App.Other.Namespace'
});
```

或者，可以在事件类前面加上一个 `.` 当使用 Echo 订阅它们时。这将允许你始终指定完全限定的类名：

```js
Echo.channel('orders')
    .listen('.Namespace\\Event\\Class', (e) => {
        //
    });
```

## 存在通道

存在通道建立在私有通道的安全性基础上，同时还暴露了意识到谁订阅了通道的附加功能。这使得构建功能强大的协作应用程序特性变得很容易，比如当其他用户正在查看同一页面时通知用户。

### 认证存在的通道

### 加入存在的通道

### 广播到存在的通道

## 客户端事件

## 通知
