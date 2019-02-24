# 安装

* [项目安装](#项目安装)
  * [服务器要求](#服务器要求)
  * [安装 Laravel](#安装-Laravel)
  * [配置](#配置)
* [Web 服务器配置](#Web-服务器配置)
  * 优雅的链接

## 项目安装

对于一个新手，[Laracasts](https://laracasts.com/series/laravel-from-scratch-2018) 针对 Laravel 框架提供了一个免费而全面的指南。这是开始您的旅程的好地方。

### 服务器要求

Laravel 框架对系统有一些要求。通过 [Laravel Homestead](https://laravel.com/docs/5.7/homestead) 虚拟机能满足这些所有的这些要求，所以强烈推荐你使用 Homestead 作为你本地的 Laravel 开发环境。

然而，如果你不使用 Homestead，你将需要确保你的服务器满足以下的要求：

* PHP >= 7.1.3
* OpenSSL PHP Extension
* PDO PHP Extension
* Mbstring PHP Extension
* Tokenizer PHP Extension
* XML PHP Extension
* Ctype PHP Extension
* JSON PHP Extension
* BCMath PHP Extension

### 安装 Laravel

Laravel 利用 [Composer](https://getcomposer.org/) 去管理它的依赖。所以，使用 Laravel 之前，确保在你的机器上安装了 Composer。

#### 通过 Laravel 安装器

首先，使用 Composer 下载 Laravel 安装器

```bash
composer global require laravel/installer
```

确保将 Composer 的系统范围供应商 bin 目录位于你的 `$PATH` 环境变量中，以便于 laravel 可执行文件就可以被系统定位。此目录根据你的操作系统存在于不同的位置；然后，一些常见的位置包括：

* `macOS`：`$HOME/.composer/vendor/bin`
* `GNU / Linux Distributions`：`$HOME/.config/composer/vendor/bin`

一旦安装过，`laravel new` 命令将在你指定的目录中创建一个全新的 Laravel 安装。例如，`laravel new blog` 将创建一个命名为 `blog` 的目录，其中包含一个已经安装的所有 Laravel 依赖项的全新 Laravel 安装：

```bash
laravel new blog
```

#### 通过 Composer 创建项目

或者，你也可以通过在你的终端中发出 Composer 的 `create-project` 命令来安装 Laravel：

```bash
composer create-project --prefer-dist laravel/laravel blog
```

#### 本地开发服务器

如果你的 PHP 安装在本地，并且你喜欢使用 PHP 的内置开发服务器去服务你的应用，你可以使用 `serve` Artisan 命令。这个命令将开启一个开发服务在 `http://localhost:8000` 上：

```bash
php artisan serve
```

通过 [Homestead](https://laravel.com/docs/5.7/homestead) 和 [Valet](https://laravel.com/docs/5.7/valet) 可以获得更强大的本地开发选择。

### 配置

## Web 服务器配置
