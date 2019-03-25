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

{% hint style="info" %}

考虑使用 [可呈现的异常](https://laravel.com/docs/5.8/errors#renderable-exceptions) 而不是在你的 `report` 方法中使用大量的 `instanceof` 检查。

{% endhint %}

#### 全局日志上下文

如果可用，Laravel 会自动将当前用户的标识作为上下文数据添加到每个异常的日志消息中。你可以通过覆盖应用程序的 `App\Exceptions\Handler` 类的 `context` 方法来定义自己的全局上下文数据。该信息将包含在通过应用程序写入的每个异常日志消息中:

```php
/**
 * 获取日志记录的默认上下文变量。
 *
 * @return array
 */
protected function context()
{
    return array_merge(parent::context(), [
        'foo' => 'bar',
    ]);
}
```

#### `report` 助手

有时你可能需要报告异常，但继续处理当前请求。`report` 助手函数允许你使用异常处理的 `report` 方法快速报告异常而不渲染错误页面：

```php
public function isValid($value)
{
    try {
        // 验证的值...
    } catch (Exception $e) {
        report($e);

        return false;
    }
}
```

#### 按类型忽略异常

异常处理器的 `$dontReport` 属性包含一个不会被记录的异常类型数组。例如，从 404 错误以及其他几种类型的错误导致的异常结果不会写入日志文件。你可以根据需要向该数组添加其他异常类型：

```php
/**
 * 不应当报告的异常类型列表。
 *
 * @var array
 */
protected $dontReport = [
    \Illuminate\Auth\AuthenticationException::class,
    \Illuminate\Auth\Access\AuthorizationException::class,
    \Symfony\Component\HttpKernel\Exception\HttpException::class,
    \Illuminate\Database\Eloquent\ModelNotFoundException::class,
    \Illuminate\Validation\ValidationException::class,
];
```

### 渲染方法

`render` 方法负责将给定的异常转换为应该发送回浏览器的 HTTP 响应。默认情况下，异常被传递给为你生成一个响应的基类。但是，你可以自由检查异常类型或返回自己自定义的响应：

```php
 /**
  * 渲染一个异常到 HTTP 响应。
  *
  * @param  \Illuminate\Http\Request  $request
  * @param  \Exception  $exception
  * @return \Illuminate\Http\Response
  */
public function render($request, Exception $exception)
{
    if ($exception instanceof CustomException) {
        return response()->view('errors.custom', [], 500);
    }

    return parent::render($request, $exception);
}
```

### 可报告 & 可渲染异常

你可以直接在自定义异常上定义 `report` 和 `render` 方法，而不是在异常处理器的 `report` 和 `render` 方法中进行类型检查异常。当存在这些方法时，框架将自动调用它们：

```php
<?php

namespace App\Exceptions;

use Exception;

class RenderException extends Exception
{
    /**
     * 报告异常。
     *
     * @return void
     */
    public function report()
    {
        //
    }

    /**
     * 渲染异常到 HTTP 响应。
     *
     * @param  \Illuminate\Http\Request
     * @return \Illuminate\Http\Response
     */
    public function render($request)
    {
        return response(...);
    }
}
```

{% hint style="info" %}

你可以键入提示 `report` 方法的任何必要依赖项，它们将通过 Laravel 的 [服务容器](https://laravel.com/docs/5.8/container) 自动注入到方法中。

{% endhint %}

## HTTP 异常

一些异常描述了来自服务器的 HTTP 错误代码。例如，这可能是『页面未找到』错误（404）、『未授权错误』（401），或者甚至是开发者生成的 500 错误。为了从应用程序中的任何地方生成这样的响应，你可以使用 `abort` 助手函数：

```php
abort(404);
```

`abort` 助手函数将立即引发异常，该异常将通过异常处理器渲染。可选地，你可以提供响应文本：

```php
abort(403, 'Unauthorized action.');
```

### 自定义 HTTP 错误页面

Laravel 可以轻松地显示为各种 HTTP 状态代码自定义的错误页面。例如，如果你希望为 404 HTTP 状态代码自定义错误页面，创建一个 `resources/views/errors/404.blade.php` 文件。该文件将服务于你的应用程序生成的所有 404 错误。该目录中的视图应该命名为与它们对应的 HTTP 状态代码相匹配。通过 `abort` 函数引发的 `HttpException`实例将作为 `$exception` 变量传递到视图：

```php
<h2>{{ $exception->getMessage() }}</h2>
```

你可以使用 `vendor:publish` Artisan 命令发布 Laravel 的错误页面模版。一旦模版被发布，你可以根据你自己的喜好自定义它们：

```php
php artisan vendor:publish --tag=laravel-errors
```
