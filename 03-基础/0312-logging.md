# 日志

## 简介

为了帮助你了解应用程序中发生的更多情况，Laravel 提供了强大的日志记录服务，允许你将消息记录到文件，系统错误日志甚至 Slack 中，以通知你的整个团队。

在后台，Laravel 利用了 [Monolog](https://github.com/Seldaek/monolog) 库，它为各种强大的日志处理器提供了支持。Laravel 使配置这些处理程序变得轻而易举，允许你混合和匹配它们来定制应用程序的日志处理。

## 配置

你的应用程序日志记录系统的所有配置都位于 `config/logging.php` 配置文件中。此文件允许你配置应用程序的日志通道，因此请确保查看每个可用的通道及其选项。我们将在下面回顾一些常见选项。

默认情况下，Laravel 在记录日志消息时将使用 `stack` 通道。`stack` 通道用于将多个日志通道聚合到一个通道中。有关构建堆栈的更多信息，请查看 [以下文档](https://laravel.com/docs/5.8/logging#building-log-stacks)。

* 配置通道名称

默认情况下，Monolog 用一个当前环境（如 `production` 或 `local`）相匹配的『通道名称』进行实例化。要改变此值，添加一个 `name` 选项到你的通道配置中：

```php
'stack' => [
    'driver' => 'stack',
    'name' => 'channel-name',
    'channels' => ['single', 'slack'],
],
```

* 可用的通道驱动

| 名称         | 描述                                                     |
| ------------ | -------------------------------------------------------- |
| `stack`      | 一个便于创建『多通道』通道的包装器                       |
| `single`     | 一个文件或路径的记录器通道（`StreamHandler`）            |
| `daily`      | 一个基于 `RotatingFileHandler` 的 Monolog 驱动，每天轮换 |
| `slack`      | 一个基于 `SlackWebhookHandler` 的 Monolog 驱动           |
| `papertrail` | 一个基于 `SyslogUdpHandler` 的 Mongolog 驱动             |
| `syslog`     | 一个基于 `SyslogHandler` 的 Mongolog 驱动                |
| `errorlog`   | 一个基于 `ErrorLogHandler` 的 Monolog 驱动               |
| `monolog`    | 一个可用于任何支持的 Monolog 处理器的 Monolog 工厂驱动   |
| `custom`     | 一个调用指定的工厂去创建通道的驱动                       |

{% hint style="info" %}

查看 [高级通道定制](https://laravel.com/docs/5.8/logging#advanced-monolog-channel-customization) 文档了解更多关于 `monolog` 和 `custom` 驱动。

{% endhint %}

* 配置单通道和每日通道

`single` 和 `daily` 通道有三个可选配置选项：`bubble`，`permission` 和 `locking`。

| 名称         | 描述                                   | 默认    |
| ------------ | -------------------------------------- | ------- |
| `bubble`     | 指示消息被处理后是否应当冒泡到其它通道 | `true`  |
| `permission` | 日志文件的权限                         | `0644`  |
| `locking`    | 在写入之前尝试锁定日志文件             | `false` |

* 配置 Papertrail 通道

`papertrail` 通道要求 `url` 和 `port` 配置选项，你可以从 [Papertrail](https://help.papertrailapp.com/kb/configuration/configuring-centralized-logging-from-php-apps/#send-events-from-php-app) 获取这些值。

* 配置 Slack 通道

`slack` 通道要求一个 `url` 配置选项。这个 URL 应当与你为 Slack 团队配置的一个 [传入 webhook](https://slack.com/apps/A0F7XDUAZ-incoming-webhooks) URL 相匹配。

### 构建日志堆栈

如前所述，`stack` 驱动程序允许你将多个通道组合成一个日志通道。为了说明如何使用日志堆栈，让我们看一下你可能在生产应用程序中看到的配置示例：

```php
'channels' => [
    'stack' => [
        'driver' => 'stack',
        'channels' => ['syslog', 'slack'],
    ],

    'syslog' => [
        'driver' => 'syslog',
        'level' => 'debug',
    ],

    'slack' => [
        'driver' => 'slack',
        'url' => env('LOG_SLACK_WEBHOOK_URL'),
        'username' => 'Laravel Log',
        'emoji' => ':boom:',
        'level' => 'critical',
    ],
],
```

让我们分析这个配置。首先，注意我们的 `stack` 通道通过 `channels` 选项：`syslog` 和 `slack` 聚合其它两个通道。因此，当记录消息时，这两个通道都将有机会去记录消息。

### 日志级别

请注意上面示例中 `syslog` 和 `slack` 通道配置中存在的 `level` 配置选项。此选项确定消息必须具有的最小『级别』才能被通道记录。 Monolog 为 Laravel 的日志记录服务提供支持，提供在 [RFC 5424 规范](https://tools.ietf.org/html/rfc5424) 中定义的所有日志级别：**emergency**，**alert**，**critical**，**error**，**warning**，，**notice**，**info** 和 **debug**。

因此，假设我们使用 `debug` 方法记录一条消息：

```php
Log::debug('An informational message.');
```

给定我们的配置，`syslog` 通道会将消息写入系统日志；但是，由于错误消息并不是 `critical` 或其级别以上，不会发送到Slack。但是，如果我们记录一个 `emergency` 消息，它将被发送到系统日志和 Slack，因为 `emergency` 级别高于两个通道设置的最低级别阈值：

```php
Log::emergency('The system is down!');
```

## 写日志消息

你可以使用 `Log` [facade](https://laravel.com/docs/5.8/facades) 写信息到日志中。如前所述，日志器提供的 8 个日志级别定义在 [RFC 5424 规范](https://tools.ietf.org/html/rfc5424)：**emergency**，**alert**，**critical**，**error**，**warning**，，**notice**，**info** 和 **debug**。

```php
Log::emergency($message);
Log::alert($message);
Log::critical($message);
Log::error($message);
Log::warning($message);
Log::notice($message);
Log::info($message);
Log::debug($message);
```

因此，你可以调用这些方法中的任何一种来记录相应级别的消息。默认情况下，消息将按照你的 `config/logging.php` 配置文件的配置写入到默认的日志通道中：

```php
<?php

namespace App\Http\Controllers;

use App\User;
use Illuminate\Support\Facades\Log;
use App\Http\Controllers\Controller;

class UserController extends Controller
{
    /**
     * 显示给定用户的资料。
     *
     * @param  int  $id
     * @return Response
     */
    public function showProfile($id)
    {
        Log::info('Showing user profile for user: '.$id);

        return view('user.profile', ['user' => User::findOrFail($id)]);
    }
}
```

**上下文信息**

还可以将一组上下文数组数据传递给日志方法。 此上下文数据将与日志消息一起被格式化并显示：

```php
Log::info('User failed to login.', ['id' => $user->id]);
```

### 写入到特定通道

有时，你可能希望将消息记录到除了应用程序默认通道之外的通道。你可以使用 `Log` facade 上的 `channel` 方法去检索并记录在你的配置文件中定义的任何通道：

```php
Log::channel('slack')->info('Something happened!');
```

如果您想要创建由多个通道组成的按需日志堆栈，可以使用 `stack` 方法：

```php
Log::stack(['single', 'slack'])->info('Something happened!');
```

## 高级的 Monlog 通道定制
