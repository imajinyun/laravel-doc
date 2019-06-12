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

## 定义广播事件

## 授权通道

## 广播事件

## 接收广播

## 在场通道

## 客户端事件

## 通知
