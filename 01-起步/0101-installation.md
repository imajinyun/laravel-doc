# 安装

* [项目安装](#xiang-mu-an-zhuang)
  * [服务器要求](#fu-wu-qi-yao-qiu)
  * [安装 Laravel](#an-zhuang-laravel)
  * [配置](#pei-zhi)
* [Web 服务器配置](#web-fu-wu-qi-pei-zhi)
  * [优雅链接](#you-ya-lian-jie)

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

通过 [Homestead](https://laravel.com/docs/5.7/homestead) 和 [Valet](https://laravel.com/docs/5.7/valet) 可以获得更为强大的本地开发选择。

### 配置

#### 公共目录

安装 Laravel 后，你应当将 Web 服务器的文档 / web 根目录目录配置为 `public` 目录。此目录中的 `index.php` 作为进入应用程序的所有 HTTP 请求的前端控制器。

#### 配置文件

Laravel 框架的所有配置文件都存储在 `config` 目录中。每一个选项进行了说明，所以你可以随意浏览文件并熟悉对你而言可用的选项。

#### 目录权限

安装 Laravel 后，你可能需要配置一些权限。`storage` 和 `bootstrap/cache` 目录下的目录应当由你的 web 服务器写入，否则 Laravel 将无法运行。如果你使用 [Homestead](https://laravel.com/docs/5.7/homestead) 虚拟机，则这些权限已被设置。

#### 应用密钥

安装 Laravel 后，你应该做的下一步是设置你的应用程序的密钥为一个随机字符串。如果你通过 Composer 或者 Laravel 安装器安装 Laravel 的，这个密钥已经通过 `php artisan key:generate` 命令为你设置了。

通常，这个字符串应当为 32 个字符长。密钥可以在 `.env` 环境文件中设置。如果你还没有将 `.env.example` 重命名为 `.env`，你应该立即处理。**如果应用密钥没有设置，则用户会话和其它加密数据将不安全！**

#### 附加配置

Laravel 几乎不需要其它配置就开箱即用。你可以自由开始开发！然而，你可能希望查看 `config/app.php` 文件及它的文档。它包含几个选项，例如，你可能根据应用程序更改 `timezone` 和 `locale` 设置。

你可能还想配置 Laravel 的一些额外组件，例如：

* [Cache](https://laravel.com/docs/5.7/cache#configuration)
* [Database](https://laravel.com/docs/5.7/database#configuration)
* [Session](https://laravel.com/docs/5.7/session#configuration)

## Web 服务器配置

### 优雅链接

#### Apache

Laravel 包含一个 `public/.htaccess` 文件，它被用来在路径中提供没有 `index.php` 前端控制器的 URL（也就是在浏览器的地址栏中隐藏 `index.php`）。在使用 Apache 服务 Laravel 之前，确保开启了 `mod_rewrite` 模块以便于 `.htaccess` 文件被服务器解析。

如果 Laravel 附带的 `.htaccess` 文件与你的 Apache 安装不能正常工作，尝试用以下的方案替代：

```bash
Options +FollowSymLinks -Indexes
RewriteEngine On

RewriteCond %{HTTP:Authorization} .
RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]

RewriteCond %{REQUEST_FILENAME} !-d
RewriteCond %{REQUEST_FILENAME} !-f
RewriteRule ^ index.php [L]
```

#### Nginx

如果你使用 Nginx，在你的站点配置中的以下指令会将所有请求定向到 `index.php` 前端控制器：

```bash
location / {
    try_files $uri $uri/ /index.php?$query_string;
}
```

当使用 [Homestead](https://laravel.com/docs/5.7/homestead) 和 [Valet](https://laravel.com/docs/5.7/valet) 时，优雅的链接将被自动配置。
