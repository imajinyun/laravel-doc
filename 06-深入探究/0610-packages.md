# 开发包

## 简介

包是向 Laravel 添加功能的主要方法。包可以是工作的像 [Carbon](https://github.com/briannesbitt/Carbon) 这样的日期的好方法，也可以是像 [Behat](https://github.com/Behat/Behat) 这样的整个 BDD 测试框架。

有不同类型的包。有些包是独立的，这意味着它们可以与任何 PHP 框架一起工作。Carbon 和 Behat 是独立包装的例子。通过在你的 `composer.json` 文件中请求，这些包中的任何一个都可以与 Laravel 一起使用。

另一方面，其他包是专门针对 Laravel 使用的。这些包可能具有专门用于增强 Laravel 应用程序的路由、控制器、视图和配置。本指南主要涵盖了那些特定于 Laravel 的包的开发。

### 关于外观的说明

在编写 Laravel 应用程序时，如果你使用契约或外观通常是无关紧要的，因为它们提供了基本相同的可测试性级别。然而，在编写包时，你的包通常不会访问 Laravel 的所有测试助手。如果希望能够像在典型的 Laravel 应用程序中一样编写你的包测试，可以使用 [Orchestral Testbench](https://github.com/orchestral/testbench) 包。

## 包发现

在 Laravel 应用程序的 `config/app.php` 配置文件中，`providers` 选项定义了应通过 Laravel 加载的服务提供者列表。当有人安装你的软件包时，你通常会希望你的服务提供商包含在此列表中。你可以在软件包的 `composer.json` 文件的 `extra` 部分中定义提供程序，而不是要求用户手动将服务提供程序添加到列表中。除了服务提供商，你还可以列出你想要注册的任何 [外观](https://laravel.com/docs/5.8/facades)：

```php
"extra": {
    "laravel": {
        "providers": [
            "Barryvdh\\Debugbar\\ServiceProvider"
        ],
        "aliases": {
            "Debugbar": "Barryvdh\\Debugbar\\Facade"
        }
    }
},
```

一旦你的包被配置为发现, Laravel 将在安装时自动注册它的服务提供者和外观，为你的包的用户创建一个方便的安装体验。

### 扩展包的发现

如果你是包的使用者，并且希望为一个包禁用包的发现功能，你可以在应用程序 `composer.json` 文件的 `extra` 部分中列出包名：

```php
"extra": {
    "laravel": {
        "dont-discover": [
            "barryvdh/laravel-debugbar"
        ]
    }
},
```

你可以使用应用程序的 `dont-discover` 指令中的 `*` 字符禁用所有包的包发现：

```php
"extra": {
    "laravel": {
        "dont-discover": [
            "*"
        ]
    }
},
```

## 服务提供者

[服务提供者](https://laravel.com/docs/5.8/providers) 是你的包和 Laravel 之间的连接点。一个服务提供者负责将内容绑定到 Laravel 的 [服务容器](https://laravel.com/docs/5.8/container) 中，并通知 Laravel 在那里加载包资源（如：视图、配置和本地化文件）。

服务提供程序扩展了 `Illuminate\Support\ServiceProvider` 类，包含两种方法：`register` 和 `boot`。基本的 `ServiceProvider` 类位于 `illuminate/support` Composer 包中，你应该将其添加到你自己的包的依赖项中。要了解有关服务提供者的结构和目的的更多信息，请查看 [它们的文档](https://laravel.com/docs/5.8/providers)。

## 资源

### 配置

通常，你需要将你的包的配置文件发布到应用程序自己的 `config` 目录。这将允许你的包的用户轻松地覆盖你默认的配置选项。要允许发布配置文件，从你的服务提供者的 `boot` 方法中调用 `publishes` 方法：

```php
/**
 * 引导任何应用程序服务。
 *
 * @return void
 */
public function boot()
{
    $this->publishes([
        __DIR__.'/path/to/config/courier.php' => config_path('courier.php'),
    ]);
}
```

现在，当你的包的用户执行 Laravel 的 `vendor:publish` 命令时，你的文件将被复制到指定的发布位置。一旦发布了配置，就可以像访问任何其他配置文件一样访问它的值：

```php
$value = config('courier.option');
```

{% hint style="danger" %}

你不应该在你的配置文件中定义闭包。当用户执行 `config:cache` Artisan 命令时，无法正确序列化它们。

{% endhint %}

#### 默认包配置

你还可以将自己的包配置文件与应用程序的已发布副本合并。这将允许你的用户仅定义他们实际上想要在已发布的配置副本中覆盖的选项。要合并配置，在你的服务提供者的 `register` 方法中使用 `mergeConfigFrom` 方法：

```php
/**
 * 在容器中注册绑定。
 *
 * @return void
 */
public function register()
{
    $this->mergeConfigFrom(
        __DIR__.'/path/to/config/courier.php', 'courier'
    );
}
```

{% hint style="danger" %}

此方法只合并配置数组的第一层。如果用户部分定义了多维配置数组，则不会合并丢失的选项。

{% endhint %}

### 路由

如果你的包包含路由，可以使用 `loadRoutesFrom` 方法加载它们。此方法将自动确定是否缓存了应用程序的路由，如果路由已经缓存，则不会加载你的路由文件：

```php
/**
 * 引导任何应用程序服务。
 *
 * @return void
 */
public function boot()
{
    $this->loadRoutesFrom(__DIR__.'/routes.php');
}
```

### 迁移

如果你的包包含 [数据库迁移](https://laravel.com/docs/5.8/migrations)，可以使用 `loadMigrationsFrom` 方法通知 Laravel 如何加载它们。`loadMigrationsFrom` 方法接受你的包迁移的路径作为其惟一参数：

```php
    /**
 * 引导任何应用程序服务。
 *
 * @return void
 */
public function boot()
{
    $this->loadMigrationsFrom(__DIR__.'/path/to/migrations');
}
```

一旦你的包的迁移已经注册，它们将在执行 `php artisan migrate` 命令时自动运行。你无需将它们导出到应用程序的主 `database/migrations` 目录。

### 翻译

如果包中包含 [翻译文件](https://laravel.com/docs/5.8/localization)，可以使用 `loadTranslationsFrom` 方法通知 Laravel 如何加载它们。例如，如果你的包名为 `courier`，你应该将以下内容添加到你的服务提供者的 `boot` 方法中：

```php
/**
 * 引导任何应用程序服务。
 *
 * @return void
 */
public function boot()
{
    $this->loadTranslationsFrom(__DIR__.'/path/to/translations', 'courier');
}
```

使用 `package::file.line` 语法约定引用包的翻译。因此，你可以像这样从 `messages` 文件中加载 `courier` 包的 `welcome` 行：

```php
echo trans('courier::messages.welcome');
```

## 命令

## 公共资产

## 发布文件组
