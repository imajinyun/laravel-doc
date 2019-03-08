# Session

## 简介

由于 HTTP 驱动的应用程序是无状态的，Session 提供一种存储有关用户跨多个请求的信息的方式。Laravel 附带了富于表现力的统一 API 访问各种会话后端。支持开箱即用的流行的后端，比如：[Memcached](https://memcached.org/)，[Redis](https://redis.io/) 和数据库。

### 配置

Session 配置文件存储在 `config/session.php`。务必查看此文件中可用的选项。默认情况下，Laravel 配置了使用 `file` 会话驱动程序，它在许多应用程序工作良好。在生产应用中程序中，你可以考虑使用 `memcached` 或 `redis` 驱动程序以获得更快的会话性能。

会话 `driver` 配置选项定义将为每个请求存储会话数据位置。Laravel 附带了几个开箱即用的优秀驱动：

* `file`：会话存储在 `storage/framework/sessions`；
* `cookie`：会话存储在安全加密的 Cookie 中；
* `database`：会话存储在关系型数据库中；
* `memcached` / `redis`：会话存储在基于这些高速的缓存的存储中的一个；
* `array`：会话存储在 PHP 数组中并将不持久化；

{% hint style="info" %}

在 [测试](https://laravel.com/docs/5.8/testing) 期间使用数组驱动，并防止会话中存储的数据被持久化。

{% endhint %}

### 驱动先决条件

#### 数据库

当使用 `database` 会话驱动程序时，你需要创建一个表来包含会话项。下面是表的示例 `Schema` 声明：

```php
Schema::create('sessions', function ($table) {
    $table->string('id')->unique();
    $table->unsignedInteger('user_id')->nullable();
    $table->string('ip_address', 45)->nullable();
    $table->text('user_agent')->nullable();
    $table->text('payload');
    $table->integer('last_activity');
});
```

你可以使用 `session:table` Artisan 命令去生成这个迁移：

```bash
php artisan session:table

php artisan migrate
```

#### Redis

在与 Laravel 一起使用 Redis 之前，你将需要通过 Composer 安装 `predis/prredis` 包（~1.0）。你可以在 `database` 配置文件中配置你的 Redis 连接。在 `session` 配置文件中，`connection` 选项可以用于指定会话使用哪个 Redis 连接。

## 使用 Session

### 检索数据

在 Laravel 中使用会话数据有两种主要的方式：全局 `session` 助手函数和通过 `Request` 实例。首先，让我们看一下通过 `Request` 实例访问会话，该实例可以在控制器方法上键入类型提示，控制器方法依赖项通过 Laravel [服务容器](https://laravel.com/docs/5.8/container) 自动注入：

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use App\Http\Controllers\Controller;

class UserController extends Controller
{
    /**
     * 显示给定用户的个人资料。
     *
     * @param  Request  $request
     * @param  int  $id
     * @return Response
     */
    public function show(Request $request, $id)
    {
        $value = $request->session()->get('key');

        //
    }
}
```

当你从会话中检索一个条目时，你还可以把一个默认值作为第二个参数传递到 `get` 方法。如果指定的键名在会话中不存在，则将返回此默认值。如果你将 `Closure` 作为默认值传递到 `get` 方法并请求的键名不存在，`Closure` 将被执行并返回它的结果：

```php
$value = $request->session()->get('key', 'default');

$value = $request->session()->get('key', function () {
    return 'default';
});
```

#### 全局会话助手

你还可以在会话中使用全局的 `session` PHP 函数去检索并存储数据。当 `session` 助手函数以单个的字符串参数调用时，它将返回会话键名的值。当助手以一个键 / 值对数组调用时，这些值将存储在会话中：

```php
Route::get('home', function () {
    // 从会话中检索一段数据...
    $value = session('key');

    // 指定一个默认值...
    $value = session('key', 'default');

    // 在会话中存储一段数据...
    session(['key' => 'value']);
});
```

{% hint style="info" %}

通过一个 HTTP 请求实例使用会话与使用全局 `session` 助手函数之间几乎没有实际区别。这两种方法都通过 `assertSessionHas` 方法 [可测试](https://laravel.com/docs/5.8/testing)，该方法适用于所有的测试用例。

{% endhint %}

#### 检索所有 Session 数据

如果要检索会话中的所有数据，你可以使用 `all` 方法：

```php
$data = $request->session()->all();
```

#### 确定一个条目是否在 Session 中存在

要确定一个条目是否在会话中存在，你可以使用 `has` 方法。如果条目存在并不为 `null` 时，`has` 方法返回 `true`：

```php
if ($request->session()->has('users')) {
    //
}
```

要确定一个条目是否在会话中存在，甚至它的值是 `null`，你可以使用 `exists` 方法。如果条目存在时，`exists` 方法返回 `true`：

```php
if ($request->session()->exists('users')) {
    //
}
```

### 存储数据

要在会话中存储数据，你通过将使用 `put` 方法或 `session` 助手函数：

```php
// 通过请求实例...
$request->session()->put('key', 'value');

// 通过全局助手函数...
session(['key' => 'value']);
```

#### 推送到数组会话值

`push` 方法可以用于推送一个新值到数组会话值中。例如，如果 `user.teams` 键包含一个团队名称的数组，你可以像这样推送一个新值到数组：

```php
$request->session()->push('user.teams', 'developers');
```

#### 检索 & 删除一个条目

`pull` 方法将在单个语句中从会话中检索并删除一个条目：

```php
$value = $request->session()->pull('key', 'default');
```

### 闪存数据

有时你可能希望仅在下一个请求中去将条目存储在会话中。你可以使用 `flash` 方法这样做。使用此方法存储在会话中的数据将仅在后续 HTTP 请求期间可用，然后将被删除。闪存数据主要适用于短期存活的状态消息：

```php
$request->session()->flash('status', 'Task was successful!');
```

如果你需要为几个请求保留闪存数据，你可以使用 `reflash` 方法，此方法将为一个额外的请求保留所有的闪存数据。如果你仅需要保留指定的闪存数据，你可以使用 `keep` 方法：

```php
$request->session()->reflash();

$request->session()->keep(['username', 'email']);
```

### 删除数据

`forget` 方法将从会话中移除一段数据。如果你想从会话中移除所有数据，你可以使用 `flush` 方法：

```php
// 忘记单个键名...
$request->session()->forget('key');

// 忘记多个键名...
$request->session()->forget(['key1', 'key2']);

$request->session()->flush();
```

### 重新生成 Session ID

重新生成会话 ID 是经常要做的，为了防止恶意用户利用 [session fixation](https://en.wikipedia.org/wiki/Session_fixation) 对你的应用程序攻击。

 如果你使用内置的 `LoginController`，Laravel 在身份认证期间自动重新生成会话 ID；然而，如果你需要手动重新生成会话 ID，你可以使用 `regenerate` 方法：

 ```php
 $request->session()->regenerate();
 ```

## 添加自定义的 Session 驱动

### 实现驱动程序

你的自定义驱动程序应该实现 `SessionHandlerInterface`。此接口只包含我们需要去实现的一些简单方法。一个存根的 MongoDB 实现看起来像这样：

```php
<?php

namespace App\Extensions;

class MongoSessionHandler implements \SessionHandlerInterface
{
    public function open($savePath, $sessionName) {}
    public function close() {}
    public function read($sessionId) {}
    public function write($sessionId, $data) {}
    public function destroy($sessionId) {}
    public function gc($lifetime) {}
}
```

{% hint style="info" %}

Laravel 不附带包含你的扩展的一个目录。你随意去放置它们到你喜欢的任何地方。在这个实例中，我们创建一个 `Extensions` 目录去放置 `MongoSessionHandler`。

{% endhint %}

由于这些方法的目的不容易理解，让我们快速介绍一下每种方法的作用：

* `open` 方法通常用于基于文件的会话存储系统。由于 Laravel 附带了一个 `file` 会话驱动，你几乎不需要在此方法中放置任何东西。你可以把它留做一个空存根。PHP 要求我们实现此方法，它实际上一个糟糕的接口设计（我们将在稍后讨论）。
* `close` 方法，像 `open` 方法一样，通常也可以忽略不计。对于大多数驱动程序，它不需要。
* `read` 方法应当返回与给定的 `$sessionId` 关联的会话数据的字符串版本。当在你的驱动程序中检索或存储会话数据时，不需要任何序列化或其它编码，因为 Laravel 将替你执行序列化。
* `write` 方法应将与 `$sessionId` 关联的给定的 `$data` 字符串写入某个持久化存储系统，比如：MongoDB，Dynamo 等等。再次，你应该不执行任何序列化——Laravel 将已经替你处理了这些。
* `destroy` 方法应该从持久化存储中移除与 `$sessionId` 关联的数据。
* `gc` 方法应该销毁比给定的 `$lifetime`还早的所有会话数据，它是一个 UNIX 时间戳。对于像 Memcached 和 Redis 这样的自助过期系统，此方法可以为留空。

### 注册驱动程序

一旦你的驱动程序实现了，你就准备在框架中注册它了。要添加额外的驱动程序到 Laravel 的会话后端，你可以在 `Session` [facade](https://laravel.com/docs/5.8/facades) 上使用 `extend` 方法。你应当从 [服务提供者](https://laravel.com/docs/5.8/providers) 的 `boot` 方法中调用 `extend` 方法。你可以从存在的 `AppServiceProvider` 或创建一个完整的新提供者来这样做：

```php
<?php

namespace App\Providers;

use App\Extensions\MongoSessionHandler;
use Illuminate\Support\Facades\Session;
use Illuminate\Support\ServiceProvider;

class SessionServiceProvider extends ServiceProvider
{
    /**
     * 执行注册后服务引导。
     *
     * @return void
     */
    public function boot()
    {
        Session::extend('mongo', function ($app) {
            // 返回 SessionHandlerInterface 接口的实现...
            return new MongoSessionHandler;
        });
    }

    /**
     * 在容器中注册绑定。
     *
     * @return void
     */
    public function register()
    {
        //
    }
}
```

一旦注册了会话驱动，你可以在你的 `config/session.php` 配置文件中使用 `mongo` 驱动程序。
