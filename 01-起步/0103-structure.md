# 目录结构

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

### tests 目录

### vendor 目录

## 应用目录

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
