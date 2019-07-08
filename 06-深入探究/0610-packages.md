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

## 资源

## 命令

## 公共资产

## 发布文件组
