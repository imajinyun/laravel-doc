# 部署

* [简介](#jian-jie)
* [服务器配置](#fu-wu-qi-pei-zhi)

## 简介

当你准备好部署你的 Laravel 应用程序到生产环境时，你可以采取一些重要的措施来确保你的应用程序尽可能高效地运行。在本文档中，我们将介绍一些好的出发点来确保你的 Laravel 应用程序恰当的部署。

## 服务器配置

### Nginx

如果你部署你的应用程序到一个运行 Nginx 的服务器上，你可以使用如下的配置文件作为配置你的 web 服务器的起点。最有可能的是，这个文件将需要根据你的服务器的配置进行定制。如果你想在管理你的服务器上需要协助，考虑使用一个像 [Laravel Forge](https://forge.laravel.com/) 的服务：

```bash
server {
    listen 80;
    server_name example.com;
    root /example.com/public;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-XSS-Protection "1; mode=block";
    add_header X-Content-Type-Options "nosniff";

    index index.html index.htm index.php;

    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    error_page 404 /index.php;

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php7.2-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```

## 优化

### 自动加载器优化

当部署到生产环境时，确保你优化 Composer 的类自动加载器映射，以便于 Composer 能快速地找到要对一个给定的类加载正确的文件：

```bash
composer install --optimize-autoloader --no-dev
```

{% hint style="info" %}

除了优化自动加载器之外，你应当总是确保在你的项目的源代码仓库中包含一个 `coomposer.lock` 文件。当 `composer.lock` 文件存在时，你的项目的依赖项能被更快地安装。

{% endhint %}

### 优化配置加载

当你将应用程序部署到生产环境时，你应当确保在你部署过程中运行 `config:cache` Artisan 命令：

```bash
php artisan config:cache
```

此命令将所有 Laravel 的配置文件合并到一个缓存文件，这次极大地减少了框架在加载配置值时必须对文件系统进行访问的次数。

{% hint style="danger" %}

如果在你部署过程中执行 `config:cache` 命令，你应当确保你仅从你的配置文件中调用 `env` 函数。一旦配置被缓存，`.env` 文件将不被加载并且对 `env` 函数的所有调用将返回 `null`。

{% endhint %}

### 优化路由加载

如果你想构建具有许多路由的大型应用程序，你应当确保在你部署的过程中运行 `route:cache` Artisan 命令：

```bash
php artisan route:cache
```

此命令将为所有路由注册缩减到一个缓存文件中的单个方法调用，从而在注册数百个路由时提高了路由注册的性能。

{% hint style="danger" %}

由于此功能使用 PHP 序列化，你仅能缓存专门使用基于控制器路由的应用程序路由。PHP 不能序列化闭包路由。

{% endhint %}

## 使用 Forge 部署

如果你还没有准备好管理自己的服务器配置，或者不熟悉配置对运行强大的 Laravel 应用程序所需的各种服务，[Laravel Forge] 是一个好的选择。

Laravel Forge 能在各种基础设施提供商（如：DigitalOcean，Linode，AWS 等等）上创建服务器。另外，Forge 安装和管理构建强大 Laravel 应用程序所需的所有工具，比如：Nginx，MySQL，Redis，Memcached，Beanstalk 等等。
