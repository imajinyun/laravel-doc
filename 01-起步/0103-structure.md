# 目录结构

* [简介](#jian-jie)
  * [模型目录在哪里](#mo-xing-mu-lu-zai-na-li)
* [根目录](#gen-mu-lu)
  * [App 目录](#app-mu-lu)
  * [Config 目录](#config-mu-lu)
  * [Bootstrap 目录](#bootstrap-mu-lu)
  * [Database 目录](#database-mu-lu)
  * [Public 目录](#public-mu-lu)
  * [Resources 目录](#resources-mu-lu)
  * [Routes 目录](#routes-mu-lu)
  * [Storage 目录](#storage-mu-lu)
  * [Tests 目录](#tests-mu-lu)
  * [Vendor 目录](#vendor-mu-lu)
* [应用目录](#ying-yong-mu-lu)
  * [Broadcasting 目录](#broadcasting-mu-lu)
  * [Console 目录](#console-mu-lu)
  * [Events 目录](#events-mu-lu)
  * [Exceptions 目录](#exceptions-mu-lu)
  * [Http 目录](#http-mu-lu)
  * [Jobs 目录](#jobs-mu-lu)
  * [Listeners 目录](#listeners-mu-lu)
  * [Mail 目录](#mail-mu-lu)
  * [Notifications 目录](#notifications-mu-lu)
  * [Policies 目录](#policies-mu-lu)
  * [Providers 目录](#providers-mu-lu)
  * [Rules 目录](#rules-mu-lu)

## 简介

默认的 Laravel 应用程序结构旨在为大型和小型应用程序提供一个很好的起点。当然，不管怎么你可以按自己的喜好随意组织你的应用程序。Laravel 对任何给定类的位置几乎没有任限制——只要 Composer 能自动加载该类。

### 模型目录在哪里

当开始使用 Laravel 时，许多开发者对缺少 `models` 目录感到困惑。然而，缺少这样的一个目录是有意的。我们发现『模型』这个词含糊不清，因为它对许多不同人的来说意味着许多不同的东西。一些开发者将应用程序的『模型』称为所有业务逻辑的总体，同时其它的开发者将『模型』称为与关系数据库交互的类。

出于这个原因，我们默认选择把 Eloquent 模型放置在 `app` 目录中，允许开发者放置他们在其它地方（如果他们选择了）。

## 根目录

### App 目录

`app` 目录包含你的应用程序的核心代码。不久我们将更加详细地探索这个目录；然而，你的应用程序中的几乎所有的类将会位于此目录中。

### Bootstrap 目录

`bootstrap` 目录包含引导框架的 `app.php` 文件。这个目录还安置一个 `cache` 目录，此目录包含为了性能优化的框架生成文件，比如：路由和服务缓存文件。

### Config 目录

顾名思义，`config` 目录包含应用程序的所有配置文件。阅读所有这些文件并熟悉所有对你而言可用的选项是一个好的主意。

### Database 目录

`database` 目录包含数据库迁移、模型工厂和填充。如果你愿意，你还可以使用此目录保存 SQLite 数据库。

### public 目录

`public` 目录包含 `index.php` 文件，该文件是进入应用程序并配置自动加载的所有请求的入口点。此目录还可安置你的资产，比如：图片，JavaScript 和 CSS。

### resources 目录

`resources` 目录包含你的视图及原始的未编译的资产，比如：LESS，SASS 或者 JavaScript。此目录也可安置你的所有语言文件。

### routes 目录

`routes` 目录包含应用程序的所有路由定义。默认情况下，Laravel 包含几个路由文件：`web.php`，`api.php`，`console.php` 和 `channels.php`。

`web.php` 文件包含 `RouteServiceProvider` 放置在 `web` 中间件组中的路由，该中间件提供会话状态，CSRF 保护和 cookie 加密。如果应用程序不提供无状态 RESTful API，则很可能在 `web.php` 文件中定义所有路由。

`api.php` 文件包含 `RouteServiceProvider` 放置在 `api` 中间件组中的路由，它提供速率限。这些路由是无状态的，因此通过这些路由进入应用程序的请求旨在经由令牌进行身份认证，并且不能访问会话状态。

### storage 目录

`storage` 目录包含你编译的 Blade 模板，基于会话的文件，缓存文件和通过框架生成的其它文件。此目录分为 `app`，`framework` 和 `logs` 目录。`app` 目录可用来存储应用程序生成的任何文件。`framework` 目录用来存储框架生成的文件和缓存。最后，`logs` 目录包含应用程序的日志文件。

`storage/app/public` 目录可用来存储像用户头像这种应该公开访问的用户生成文件。你应当在 `public/storage` 处创建一个指向此目录的符号链接。你可以使用 `php artisan storage:link` 命令创建此链接。

### tests 目录

`tests` 目录包含你的自动化测试。一个开箱即用的示例 [PHPUnit](https://phpunit.de/) 测试被提供。每个测试类应当以单词 `Test` 为后缀。你可以使用 `phpunit` 或者 `php vendor/bin/phpunit` 命令运行测试。

### vendor 目录

`vendor` 目录包含 [Composer](https://getcomposer.org/) 依赖项。

## 应用目录

你的大部分的应用程序都位于 `app` 目录中。默认情况下，此目录的命名空间为 `App`，并通过 Composer 使用 [PSR-4 autoloading standard](https://www.php-fig.org/psr/psr-4/) 自动加载。

`app` 目录包含额外的各种目录，比如：`Console`，`Http` 和 `Providers`。将 `Console` 和 `Http` 目录视为向应用程序的核心提供 API。HTTP 协议和 CLI 都是与应用程序交互的机制，但实际上并不包含应用程序逻辑。换句话说，它们是向你的应用程序发出命令的两种方式。`Console` 目录包含所有的 Artisan 命令，而 `Http` 目录包含你的控制器，中间件和请求。

当你使用 `make` Artisan 命令生成类时，将在 `app` 目录下生成各种其它目录。因此，例如，`app/Jobs` 目录直到你执行 `make:job` Artisan 命令去生成一个作业类之前将不存在。

{% hint style="info" %}

许多类通过 Artisan 命令生成在 `app` 目录中。为了查看可用的命令，在你的终端运行 `php artisan list make` 命令。

{% endhint %}

### Broadcasting 目录

### Console 目录

### Events 目录

### Exceptions 目录

### Http 目录

### Jobs 目录

### Listeners 目录

### Mail 目录

### Notifications 目录

### Policies 目录

### Providers 目录

### Rules 目录
