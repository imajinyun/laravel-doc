# 认证

## 简介

{% hint style="info" %}

**想快速入门？** 在一个全新的 Laravel 应用程序中只需运行 `php artisan make:auth` 和 `php artisan migrate`。然后，将浏览器导航到 `http：//your-app.test/register` 或分配给你的应用程序的任何其他 URL。这两个命令将负责整个认证系统的脚手架！

{% endhint %}

Laravel 使实现身份验证变得非常简单。事实上，几乎所有东西都是开箱即用的。认证配置文件位于 `config/auth.php`，其中包含几个文档友好的选项，用于调整认证服务的行为。

Laravel 的认证设施的核心是『守卫』和『提供者』。守卫定义了如何为每个请求对用户进行认证。例如：Laravel 附带一个 `session` 防护，它使用会话存储和 `cookie` 来维护状态。

提供者定义如何从持久存储中检索用户。 Laravel 支持使用 Eloquent 和数据库查询构建器检索用户。但是，你可以根据应用程序的需要自由定义其他提供者。

如果这一切听起来有点混乱，请不要担心！许多应用程序永远不需要修改默认的认证配置。

### 数据库注意事项

默认情况下，Laravel 在你的 `app` 目录中包含一个 `App\User` [Eloquent 模型](https://laravel.com/docs/5.8/eloquent)。该模型可与默认的 Eloquent 认证驱动程序一起使用。如果你的应用程序未使用 Eloquent，则可以使用使用 Laravel 查询构建器的 `database` 认证驱动程序。

## 认证快速入门

## 手动认证用户

## HTTP 基础认证

## 注销

## 社交认证

## 添加自定义防护

## 添加自定义用户提供者

## 事件
