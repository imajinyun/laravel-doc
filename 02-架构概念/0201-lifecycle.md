# 请求周期

* [简介](#jian-jie)
* [生命周期概述](#sheng-ming-zhou-qi-gai-shu)
* [专注于服务提供者](#zhuan-zhu-yu-fu-wu-ti-gong-zhe)

## 简介

当在『真实世界』使用任何工具时，如果你理解该工具的工作原理，你会感到更加自信。应用程序开发也不例外。当你理解你的开发工具的功能时，你会感觉使用它们更加舒服和自信。

本文档是目标是给你一个 Laravel 框架是如何工作的良好，高级概述。通过更好地了解整体的框架，一切感觉不在那么『神奇』并将更加自信的构建你的应用程序。如果你不立即理解所有的术语，不要灰心！仅尝试基本理解正在发生的事情，并且在你探索文档的其它部分时你的知识将会增长。

## 生命周期概述

### 首要的事情

对 Laravel 应用程序所有的请求入口点是 `public/index.php` 文件。所有的请求通过你的 web 服务器（Apache / Nginx）配置定向到这个文件。`index.php` 文件不包含太多的代码。相反，它是加载框架其余部分的起始点。

`index.php` 文件加载 Composer 生成的自动加载器定义，然后从 `bootstrap/app.php` 脚本中检索 Laravel 应用程序的一个实例。Laravel 本身采取的第一个行动是创建一个应用程序 / [服务容器](https://laravel.com/docs/5.8/container) 实例。

### Http / Console 内核

接下来，传入的请求发送到 HTTP 内核或控制台内核，具体取决于进入应用程序的请求类型。这两个内核充当所有请求流经的中心位置。现在，让我们聚集 HTTP 内核，它位置 `app/Http/Kernel.php` 中。

HTTP 内核继承 `Illuminate\Foundation\Http\Kernel` 类，它定义了一个请求执行之前运行的 `bootstrappers` 数组。这些引导配置错误处理，配置日志，[检测应用环境](https://laravel.com/docs/5.8/configuration#environment-configuration)，并执行在实际处理请求之前需要完成的其它任务。

HTTP 内核也定义了一个 HTTP [中间件](https://laravel.com/docs/5.8/middleware) 列表，所有请求必须在应用程序处理之前通过。这些中间件处理读取和写入 [HTTP 会话](https://laravel.com/docs/5.8/session)，确定应用程序是否处于维护模式，[验证 CSRF 令牌](https://laravel.com/docs/5.8/csrf) 等等。

HTTP 内核的 `handle` 方法的方法签名相当简单：接受一个 `Request` 并返回一个 `Response`。将内核视为代表整个应用程序的大黑盒子。提供 HTTP 请求并将返回 HTTP 响应。

#### 服务提供者

最重要的内核引导操作之一是为你的应用程序加载 [服务提供者](https://laravel.com/docs/5.8/providers)。应用程序的所有服务提供者都配置在 `config/app.php` 配置文件的 `providers` 数组中。首先，将在所有的提供者上调用 `register` 方法，然后，一旦所有的提供者都已注册，`boot` 方法将被调用。

服务提供者是负责引导所有框架中的各种组件，例如，数据库，队列，验证和路由组件。由于它们引导和配置框架提供的每个功能，因此服务提供者是整个 Laravel 引导程序过程中最重要的方面。

#### 调试请求

一旦应用程序被引导并且所有服务提供者都已注册，`Request` 将被切换到路由器进行调度。路由器将请求分发到路由或者控制器，以及运行任何路由的特定的中间件。

## 专注于服务提供者

服务提供者确实是引导 Laravel 应用程序的关键。创建应用程序实例，注册服务提供者，并将请求传递到引导的应用程序。它真的很简单！

牢牢掌握如何通过服务提供者构建和引导 Laravel 应用程序是非常有价值的。你的应用程序的默认服务提供者存储在 `app/Providers` 目录中。

默认情况下，`AppServiceProvider` 是完全空的。此提供者是添加你的应用程序自己的引导和服务容器绑定的好地方。对于大型应用，你可能希望创建多个服务提供者，每个服务都具有更细粒度的引导类型。
