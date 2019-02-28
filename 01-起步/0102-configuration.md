# 配置

* [概述](#gai-shu)
* [环境配置](#huan-jing-pei-zhi)
  * [环境变量类型](#huan-jing-bian-liang-lei-xing)
  * [环境变量类型](#jian-suo-huan-jing-pei-zhi)
  * [确定当前环境](#que-ding-dang-qian-huan-jing)
* [访问配置值](#fang-wen-pei-zhi-zhi)
* [配置缓存](#pei-zhi-huan-cun)
* [维护模式](#wei-hu-mo-shi)

## 概述

Laravel 框架的所有配置文件都存储在 `config` 目录中。每一个选项进行了说明，所以你可以随意浏览文件并熟悉对你而言可用的选项。

## 环境配置

基于应用程序运行的的环境有不同的配置值，通常是有用的。例如，你可能希望在你的生产服务器上使用一个不同的于本地的缓存驱动。

为了实现这一点，Laravel 利用了 Vance Lucas 写的 [DotEnv](https://github.com/vlucas/phpdotenv) PHP 库。在一个全新的 Laravel 安装中，在你的应用的根目录下包含一个 `.env.example` 文件。如果你通过 Composer 安装 Laravel，这个文件将自动被重命名为 `.env`。否则，你应当手动签名这个文件。

你的 `.env` 文件不应该提交到你的应用程序的源代码系统中，由于使用你的应用程序的每个开发者 / 服务器可能需要一个不同的环境配置。此外，如果入侵者获取你的源代码控制仓库的访问权限，这将是一个安全风险，因为任何敏感凭证将被显露。

如果你正在与团队一起开发，你可能希望继续在你的应用程序中去包含一个 `.env.example` 文件。通过在示例配置中放置占位符值，团队中的其它开发者可以清楚看到运行应用程序所需的环境变量。你也可以创建一个 `.env.testing` 文件。当运行 PHPUnit 测试或者使用 `--env=testing` 选项执行 Artisan 命令时该文件将覆盖 `.env` 文件。

{% hint style="info" %}

在你的 `.env` 文件中的任何变量可以通过外部环境变量被覆盖，比如服务器级别和系统级别的环境变量。

{% endhint %}

### 环境变量类型

在你的 `.env` 文件中的所有变量被解析为字符串，因此创建一些保留值以允许你从 `env()` 函数返回一个更宽泛的类型：

| `.env` 值 | `env()` 值  |
| --------- | ----------- |
| true      | (bool)true  |
| (true)    | (bool)true  |
| false     | (bool)false |
| (false)   | (bool)false |
| empty     | (string)''  |
| (empty)   | (string)''  |
| null      | (null)null  |
| (null)    | (null)null  |

如果你需要一个包含空格的值去定义环境变量，你可以通过将值包在双引号中来实现。

```php
APP_NAME="My Application"
```

### 检索环境配置

当你的应用接受一个请求时，此文件列出的所有变量都将加载到 PHP 的超全局变量 `$_ENV` 中。无论如何，你可以使用 `env` 助手函数从配置文件的这些变量中去检索值。实际上，如果你查看 Laravel 的配置文件，你将注意到已经使用此助手函数的几个选项：

```php
'debug' => env('APP_DEBUG', false),
```

第二个传递到 `env` 函数的是『默认值』。如果给定键在环境变量中不存在，这个值将被使用。

### 确定当前环境

从你的 `.env` 文件中通过 `APP_ENV` 变量确定当前应用程序的环境。你可以通过 `App` [facade](https://laravel.com/docs/5.7/facades) 上的 `environment` 方法访问此值：

```php
$environment = App::environment();
```

你也可以传递参数到 `environment` 方法以检查环境是否与给定的值匹配。如果环境匹配任何给定的值，这个方法将返回 `true`：

```php
if (App::environment('local')) {
    // The environment is local
}

if (App::environment(['local', 'staging'])) {
    // The environment is either local OR staging...
}
```

{% hint style="info" %}

当前应用程序环境检测可以被一个服务器级别的 `APP_ENV` 环境变量覆盖。当你需要为不同的环境配置共享相同的应用程序时，这将非常有用，因此你可以在你服务器的配置里设置一个给定的主机以匹配给定的环境。

{% endhint %}

### 从调试页面隐藏环境变量

当一个异常未被捕获并且 `APP_DEBUG` 环境变量是 `true`，调试页面将显示所有环境变量及其内容。在某些情况下你可能想要隐藏某些变量。你可以在你的 `config/app.php` 配置文件中通过更新 `debug_blacklist` 选项来完成此操作。

在环境变量和服务器 / 请求数据中有一些变量是可用的。因此，你可能需要将它们列入到 `$_ENV` 和 `$_SERVER` 的黑名单中：

```php
return [

    // ...

    'debug_blacklist' => [
        '_ENV' => [
            'APP_KEY',
            'DB_PASSWORD',
        ],

        '_SERVER' => [
            'APP_KEY',
            'DB_PASSWORD',
        ],

        '_POST' => [
            'password',
        ],
    ],
];
```

## 访问配置值

你可以从应用程序的任何位置使用全局 `config` 助手函数轻松访问你的配置值。此配置值可以使用 『.』语法访问，该语法包含你要访问的文件名称和选项。还可以指定一个默认的值，如果配置选项不存在，将返回默认值：

```php
$value = config('app.timezone');
```

在运行时设置配置值，将一个数组传递到 `config` 助手函数：

```php
config(['app.timezone' => 'America/Chicago']);
```

## 配置缓存

为了提高应用程序的速度，你应该使用 `config:cache` Artisan 命令将所有配置文件缓存到单个文件中。这将应用程序的所有配置选项组合到一个文件中，该文件将通过框架快速加载。

通常你应该运行 `php artisan config:cache` 命令作为你生产部署例程的一部分。这个命令不应该运行在本地开发中，因为在应用程序开发期间中配置选项将频繁的需要去更改。

{% hint style="danger" %}

如果你在部署过程中执行 `config:cache` 命令，你应该确保只从配置文件中调用 `env` 函数。一旦缓存被缓存，`.env` 文件将不被加载，并且所有对 `env` 函数的调用将返回 `null`。

{% endhint %}

## 维护模式

当你的应用程序处于维护模式时，进入应用程序的所有请求将显示一个自定义视图。这使得更新或者当执行维护时轻松『禁用』应用程序。维护模式检查程序包含在应用程序的默认中间件堆栈中。如果应用程序处于维护模式，则一个状态码为 503 的 `MaintenanceModeException` 将被抛出。

要启用维护模式，执行 `down` Artisan 命令：

```bash
php artisan down
```

你还可以向 `down` 命令提供 `message` 和 `retry` 选项。`message` 值可用于显示或记录自定义消息，而 `retry` 值将作为 `Retry-After` 的 HTTP 头值被设置：

```bash
php artisan down --message="Upgrading Database" --retry=60
```

即使在维护模式下，也可以使用此命令的 `allow` 选项让特定的 IP 地址或网络允许访问应用程序：

```bash
php artisan down --allow=127.0.0.1 --allow=192.168.0.0/16
```

要禁用维护模式，请使用 `up` 命令：

```bash
php artisan up
```

{% hint style="info" %}

你可以通过在 `resources/views/errors/503.blade.php` 文件中定义你自己模板来自定义默认的维护模式模板。

{% endhint %}

### 维护模式 & 队列

当你的应用程序处于维护模式时，[队列作业](https://laravel.com/docs/5.7/queues) 将不会处理。一旦应用程序退出维护模式时，作业将继续正常处理。

### 维护模式的替代方案

由于维护模式要求你的应用程序有几秒钟的停机，因此请考虑像 [Envoyer](https://envoyer.io/) 这样的替代方案来完成 Laravel 的零停机部署。
