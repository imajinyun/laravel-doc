# Redis

## 简介

[Redis](https://redis.io/) 是一个开源的先进键值存储。它通常被称为数据结构服务器，因为键可以包含 [strings](https://redis.io/topics/data-types#strings)、[hashes](https://redis.io/topics/data-types#hashes)、[lists](https://redis.io/topics/data-types#lists)、[sets](https://redis.io/topics/data-types#sets) 和 [sorted sets](https://redis.io/topics/data-types#sorted-sets)。

在将 Redis 与 Laravel 一起使用之前，你需要通过 Composer 安装 `predis/predis` 软件包：

```bash
composer require predis/predis
```

或者，你可以通过 PECL 安装 [PhpRedis](https://github.com/phpredis/phpredis) PHP 扩展。该扩展安装起来更加复杂，但对于大量使用 Redis 的应用程序，它可能会带来更好的性能。

### 配置

应用程序的 Redis 配置位于 `config/database.php` 配置文件中。在此文件中，你将看到包含你的应用程序使用的 Redis 服务器的一个 `redis` 数组：

```php
'redis' => [

    'client' => 'predis',

    'default' => [
        'host' => env('REDIS_HOST', '127.0.0.1'),
        'password' => env('REDIS_PASSWORD', null),
        'port' => env('REDIS_PORT', 6379),
        'database' => env('REDIS_DB', 0),
    ],

    'cache' => [
        'host' => env('REDIS_HOST', '127.0.0.1'),
        'password' => env('REDIS_PASSWORD', null),
        'port' => env('REDIS_PORT', 6379),
        'database' => env('REDIS_CACHE_DB', 1),
    ],

],
```

默认的服务器配置应该足以进行开发。但是，你可以根据你的环境自由地修改这个数组。在你的配置文件中定义的每个 Redis 服务器都需要有一个名称、主机和端口。

#### 配置集群

如果你的应用程序正在使用一个 Redis 服务器集群，那么你应该在你的一个 Redis 配置 `clusters` 键中定义这些集群：

```php
'redis' => [

    'client' => 'predis',

    'clusters' => [
        'default' => [
            [
                'host' => env('REDIS_HOST', 'localhost'),
                'password' => env('REDIS_PASSWORD', null),
                'port' => env('REDIS_PORT', 6379),
                'database' => 0,
            ],
        ],
    ],

],
```

默认情况下，集群将跨节点执行客户端分片，允许你共享节点并创建大量可用 RAM。但是，请注意，客户端分片不处理故障转移；因此，主要适用于从另一个主数据存储中可用的缓存数据。如果希望使用本机 Redis 集群，你应该在 Redis 配置的 `options` 键中指定：

```php
'redis' => [

    'client' => 'predis',

    'options' => [
        'cluster' => 'redis',
    ],

    'clusters' => [
        // ...
    ],

],
```

### Predis

除默认 `host`、`port`、`database` 和 `password` 服务器配置选项外，Predis 还支持可为每个 Redis 服务器定义的其他 [连接参数](https://github.com/nrk/predis/wiki/Connection-Parameters)。 要使用这些附加配置选项，将它们添加到 `config/database.php` 配置文件中的你的 Redis 服务器配置中：

```php
'default' => [
    'host' => env('REDIS_HOST', 'localhost'),
    'password' => env('REDIS_PASSWORD', null),
    'port' => env('REDIS_PORT', 6379),
    'database' => 0,
    'read_write_timeout' => 60,
],
```

### PhpRedis

要使用 PhpRedis 扩展，你应该将 Redis 配置的 `client` 选项更改为 `phpredis`。你可以在 `config/database.php` 配置文件中找到此选项：

```php
'redis' => [

    'client' => 'phpredis',

    // 其余的 Redis 配置...
],
```

除了默认的 `host`、`port`、`database` 和 `password` 服务器配置选项外，PhpRedis 还支持以下附加连接参数：`persistent`、`prefix`、`read_timeout` 和 `timeout`。你可以在 `config/database.php` 配置文件中将以下任何选项添加到你的 Redis 服务器配置中：

```php
'default' => [
    'host' => env('REDIS_HOST', 'localhost'),
    'password' => env('REDIS_PASSWORD', null),
    'port' => env('REDIS_PORT', 6379),
    'database' => 0,
    'read_timeout' => 60,
],
```

## 与 Redis 交互

你可以通过调用 Redis [外观](https://laravel.com/docs/5.8/facades) 上的各种方法与 Redis 进行交互。Redis 外观支持动态方法，这意味着你可以在外观上调用任何 [Redis 命令](https://redis.io/commands)，该命令将直接传递给 Redis。在本例中，我们将通过调用 `Redis` 外观上的 `get` 方法来调用 Redis 的 `GET` 命令：

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use Illuminate\Support\Facades\Redis;

class UserController extends Controller
{
    /**
     * 显示给定用户的个人资料。
     *
     * @param  int  $id
     * @return Response
     */
    public function showProfile($id)
    {
        $user = Redis::get('user:profile:'.$id);

        return view('user.profile', ['user' => $user]);
    }
}
```

如上所述，你可以在 Redis 外观上调用任何 Redis 命令。Laravel 使用魔术方法将命令传递给 Redis 服务器，因此传递 Redis 命令期望的参数：

```php
Redis::set('name', 'Taylor');

$values = Redis::lrange('names', 5, 10);
```

或者，你也可以使用 `command` 方法将命令传递给服务器，该方法接受命令的名称作为第一个参数，并将值数组作为第二个参数：

```php
$values = Redis::command('lrange', ['name', 5, 10]);
```

#### 使用多个 Redis 连接

你可以通过调用 `Redis::connection` 方法获得一个 Redis 实例：

```php
$redis = Redis::connection();
```

这将为你提供缺省 Redis 服务器的实例。你还可以将连接或集群名称传递给 `connection` 方法，以获得在你的 Redis 配置中定义的特定服务器或集群：

```php
$redis = Redis::connection('my-connection');
```

### 管道命令

当你需要在一次操作中向服务器发送许多命令时，应该使用管道。`pipeline` 方法接受一个参数：一个接收 Redis 实例的 `Closure`。你可以向这个 Redis 实例发出你的所有命令，它们都将在一个操作中执行：

```php
Redis::pipeline(function ($pipe) {
    for ($i = 0; $i < 1000; $i++) {
        $pipe->set("key:$i", $i);
    }
});
```

## 发布 / 订阅
