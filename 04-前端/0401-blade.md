# Blade 模板

## 简介

Blade 是 Laravel 提供的简单而强大的模板引擎。不像其它流行的 PHP 模板引擎，Blade 不限制你在视图中使用原生 PHP 代码。事实上，所有的 Blade 视图被编译成原生 PHP 代码并缓存直到被修改，这意味着对应用的性能而言 Blade 基本上是零开销。Blade 的视图文件使用 `.blade.php` 作为文件扩展名，一般存储在 `resources/views` 目录下。

## 模版继承

### 定义一个布局

使用 Blade 的两个主要好处是模板的 *inheritance* 和 *sections*。起步，让我们看一个简单的例子。首先，我们将检查『master』页面布局。由于大多数 web 应用在不同的页面上保持相同的总体布局，因此可以方便的将此布局定义为单个 Blade 视图：

```php
<!-- 存储在 resources/views/layouts/app.blade.php -->

<html>
    <head>
        <title>App Name - @yield('title')</title>
    </head>
    <body>
        @section('sidebar')
            This is the master sidebar.
        @show

        <div class="container">
            @yield('content')
        </div>
    </body>
</html>
```

如你所见，这个文件包含典型的 HTML 标记。然而，注意 `@section` 和 `@yield` 指令。顾名思义，`@section` 指令定义了一部分内容，而 `@yield` 指令被用来显示给定部分的内容。

现在我们已经为应用程序定义了一个布局，让我们定义一个继承此布局的子页面。

### 扩展一个布局

定义一个子视图时，使用 Blade `@extends` 指令去指定子视图应『inherit』的布局。扩展 Blade 视图的布局可以使用 `@section` 指令将内容注入布局的某部分中。记住，如上所述，这些部分的内容将使用 `@yield` 指令显示在布局中：

```php
<!-- 存储在 resources/views/child.blade.php -->

@extends('layouts.app')

@section('title', 'Page Title')

@section('sidebar')
    @parent

    <p>This is appended to the master sidebar.</p>
@endsection

@section('content')
    <p>This is my body content.</p>
@endsection
```

这个实例中，`sidebar` 部分利用 `@parent` 指令将内容附加（而不是覆盖）到布局的侧边栏中。当视图被渲染时，`@parent` 指令将被布局的内容替换掉。

{% hint style="info" %}

与之前的实例相反，此 `sidebar` 部分以 `@endsection` 替代 `@show` 结尾。`@endsection` 指令仅定义一个部分，而 `@show` 将定义并 **立即 `yield`** 该部分。

{% endhint %}

可以使用全局 `view` 助手函数从路由返回 Blade 视图：

```php
Route::get('blade', function () {
    return view('child');
});
```

## 组件 & 插槽

## 显示数据

## 控制结构

## 表单

## 包含子视图

## 堆栈

## 服务注入

## 扩展 Blade
