# 配置

* [概述](#gai-shu)
* [环境配置](#huan-jing-pei-zhi)
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

{% endhint %

## 访问配置值

## 配置缓存

## 维护模式
