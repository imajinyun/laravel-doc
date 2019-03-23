# 错误处理

## 简介

当你开启一个新的 Laravel 项目时，已经为你配置了错误和异常处理。 `App\Exceptions\Handler` 类是记录通过应用程序触发的所有异常然后渲染到用户的地方。在本文档中，我们将更深入地探讨这个类。

## 配置

你的 `config/app.php` 配置文件中的 `debug` 选项决定了关于一个错误的多少信息实际显示到用户。默认情况下，此选项遵循 `APP_DEBUG` 环境变量的值，它存储在你的 `.env` 文件中。

在本地开发环境，你应当设置 `APP_DEBUG` 环境变量为 `true`。在你的生产环境，此值应当始终为 `false`。如果该值在生产环境中设置为 `true`，你冒险暴露敏感配置值到你的应用程序的终端用户。

## 异常处理

### 报告方法

所有异常通过 `App\Exceptions\Handler` 类处理。此类包含两个方法：`report` 和 `render`。我们将详细检查这些方法。`report` 方法用于记录异常或发送它们到像 [Bugsnag](https://bugsnag.com/) 或 [Sentry](https://github.com/getsentry/sentry-laravel) 的外部服务。默认情况下，`report` 方法传递异常到记录异常到基类。然而，你可以随意根据你的需要记录异常。

例如，如果需要以不同的方式报告不同类型的异常，可以使用 PHP 的 `instanceof` 对照运算符：

```php
/**
 * 报告或记录一个异常。
 *
 * 这是一个发送异常到 Sentry，Bugsnag 等的好地方。
 *
 * @param  \Exception  $exception
 * @return void
 */
public function report(Exception $exception)
{
    if ($exception instanceof CustomException) {
        //
    }

    parent::report($exception);
}
```

## HTTP 异常

### 自定义 HTTP 错误页面
