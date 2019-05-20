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

为 `App\User` 模型构建数据库模式时，请确保密码列的长度至少为 60 个字符。维护 255 个字符的默认字符串列长度是一个不错的选择。

此外，你应该验证你的 `user`（或相当的）表是否包含 100 个字符的可空字符串 `remember_token` 列。此列将用于为登录应用程序时选择『记住我』选项的用户存储令牌。

## 认证快速入门

Laravel 附带了几个预构建的认证控制器，它们位于 `App\Http\Controllers\Auth` 命名空间中。`RegisterController` 处理新用户注册，`LoginController` 处理认证，`ForgotPasswordController` 处理用于重置密码的电子邮件链接，`ResetPasswordController` 包含重置密码的逻辑。这些控制器中的每一个都使用特性来包括它们必要的方法。对于许多应用程序，你根本不需要修改这些控制器。

### 路由

Laravel 提供了一种快速方法，可以使用一个简单的命令来支持你进行认证所需的所有路由和视图：

```bash
php artisan make:auth
```

此命令应当在新应用程序上使用，并将安装布局视图，注册和登录视图以及所有认证端点的路由。还将生成一个 `HomeController` 来处理登录后对应用程序仪表盘的请求。

{% hint style="info" %}

如果你的应用程序不需要注册，我可以通过移除新创建的 `RegisterController` 并修改你的路由定义 `Auth::routes(['register' => false]);` 来禁用它。

{% endhint %}

### 视图

如前所述，`php artisan make:auth` 命令将创建认证需要的所有视图并将它们放置在 `resources/views/auth` 目录下。

`make:auth` 命令还将为应用程序创建一个包含基布局的 `resources/views/layouts` 目录。所有这些视图都使用 Bootstrap CSS 框架，但是你可以随意自定义它们。

### 身份认证

现在你已为所包含的认证控制器设置了路由和视图，你可以为应用程序注册和验证新用户！你可以在浏览器中访问你的应用程序，因为认证控制器已经包含逻辑（通过其特性）来验证现有用户并将新用户存储在数据库中。

#### 路径自定义

#### 用户名自定义

#### 守卫自定义

#### 验证 / 存储自定义

### 检索认证的用户

### 保护路由

### 登录限制

## 手动认证用户

## HTTP 基础认证

## 注销

## 社交认证

## 添加自定义防护

## 添加自定义用户提供者

## 事件
