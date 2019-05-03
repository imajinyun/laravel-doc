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

组件和插槽为部分和布局提供了类似的好处；然而，有些人发现组件和插槽的心理模型更容易理解。首先，让我们想像一下希望在整个应用程序中可重用的『alert』组件：

```php
<!-- /resources/views/alert.blade.php -->

<div class="alert alert-danger">
    {{ $slot }}
</div>
```

`{{ $slot }}` 变量将包含我们希望注入到组件的内容。现在，去构造这个组件，我们可以使用 Blade 的 `@component` 指令：

```php
@component('alert')
    <strong>Whoops!</strong> Something went wrong!
@endcomponent
```

有时为组件定义多个插槽会很有帮助。让我们修改我们的警告组件以允许注入一个『标题』。命名的插槽可以通过『回显』与它们的名称相匹配的变量来显示：

```php
<!-- /resources/views/alert.blade.php -->

<div class="alert alert-danger">
    <div class="alert-title">{{ $title }}</div>

    {{ $slot }}
</div>
```

现在，我们能使用 `@slot` 指令注入内容到命名的插槽中。任何不在 `@slot` 指令内的内容都将被传递给 `@slot` 变量中的组件：

```php
@component('alert')
    @slot('title')
        Forbidden
    @endslot

    You are not allowed to access this resource!
@endcomponent
```

### 传递额外的数据到组件

有时你需要传递额外的数据到组件。为了这个原因，你可能传递一个数组数据作为第二个参数到 `@compoent` 指令。所有数据都将作为变量提供给组件模板：

```php
@component('alert', ['foo' => 'bar'])
    ...
@endcomponent
```

### 别名组件

如果你的 Blade 组件存储在一个子目录中，你可能希望为了容易访问他们而添加别名。例如，想像一个 Blade 组件被存储在 `resources/views/components/alert.blade.php`。你可以使用 `component` 方法将 `components.alert` 组件别名为 `alert`。通常，这应该在 `AppServiceProvider` 类的 `boot` 方法中去做：

```php
use Illuminate\Support\Facades\Blade;

Blade::component('components.alert', 'alert');
```

一旦组件被别名化，你可以使用一个指令来渲染它：

```php
@alert(['type' => 'danger'])
    You are not allowed to access this resource!
@endalert
```

如果组件没有额外的插槽，你可以忽略组件的参数：

```php
@alert
    You are not allowed to access this resource!
@endalert
```

## 显示数据

你也可以通过将变量包裹花括号中来显示传递到 Blade 视图的数据。例如，给定以下的路由：

```php
Route::get('greeting', function () {
    return view('welcome', ['name' => 'Samantha']);
});
```

你可以显示 `name` 变量的内容像这样：

```php
Hello, {{ $name }}.
```

{% hint style="info" %}
Blade `{{ }}` 语句将通过 PHP 的 `htmlspecialchars` 函数自动发送，以防止 XSS 攻击。
{% endhint %}

不局限于显示传递给视图的变量内容。你也可以显示任何 PHP 函数的结果。实际上，你可以在 Blade 显示语句中放置你希望的任何 PHP 代码：

```php
The current UNIX timestamp is {{ time() }}.
```

### 显示未转义的数据

默认情况下，Blade `{{ }}` 语句将通过 PHP 的 `htmlspecialchars` 函数自动发送以阻止 XSS 攻击。如果你不想你的数据被转义，你可以使用以下的语法：

```php
Hello, {!! $name !!}.
```

{% hint style="info" %}
当显示应用程序用户提供的内容时要非常小心。在显示用户提供的数据时，始终使用转义的双花括号来防止 XSS 攻击。
{% endhint %}

### 渲染 JSON 数据

有时你可以将数组传递到你的视图，以打算将其渲染作为 JSON 以初始化一个 JavaScript 变量。例如：

```php
<script>
    var app = <?php echo json_encode($array); ?>;
</script>
```

然而，你可以使用 Blade 的 `@json` 指令，而不是手动调用 `json_encode`：

```php
<script>
    var app = @json($array);
</script>
```

`@json` 指令对于接入 Vue 组件或 `data-*` 属性也是很有用的：

```php
<example-component :some-prop='@json($array)'></example-component>
```

{% hint style="danger" %}
在元素属性中使用 `@json` 需要用单引号括起来。
{% endhint %}

### HTML 实体编码

默认情况下，Blade（和 Laravel `e` 帮助程序）将对 HTML 实体进行双重编码。如果要禁用双重编码，请从 `AppServiceProvider` 的 `boot` 方法中调用 `Blade::withoutDoubleEncoding` 方法：

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Blade;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * 引导任何应用程序服务。
     *
     * @return void
     */
    public function boot()
    {
        Blade::withoutDoubleEncoding();
    }
}
```

### Blade & JavaScript 框架

由于许多 JavaScript 框架也使用『花』括号来表示应在浏览器中显示给定的表达式，你也可以使用 `@` 符号去通知 Blade 渲染引擎一个表达式应当保持不变。例如：

```php
<h1>Laravel</h1>

Hello, @{{ name }}.
```

在这个实例中，符号 `@` 将被 Blade 移除；然而，`{{ name }}` 表达式将仍然不受 Blade 引擎的影响，允许它由 JavaScript 框架去渲染。

### 指令 @verbatim

如果你在你的模板大部分中显示 JavaScript 变量，你可以将 HTML 包裹在 `@verbatim` 指令中，以便于你不必为每个 Blade 显示语句添加一个 `@` 符号：

```php
@verbatim
    <div class="container">
        Hello, {{ name }}.
    </div>
@endverbatim
```

## 控制结构

除了模板继承和显示数据外，Blade 还为常见的 PHP 控制结构（如条件语句和循环）提供了方便的快捷方式。这些快捷方式提供了一个非常干净，简洁的 PHP 控制结构工作方式，同时也保留了对 PHP 对应的相似性。

### IF 语句

你可以构造 `if` 语句使用 `@if`，`@elseif`，`@else`，`endif` 指令，这些指令的功能与它们的 PHP 对应程序完全相同：

```php
@if (count($records) === 1)
    I have one record!
@elseif (count($records) > 1)
    I have multiple records!
@else
    I don't have any records!
@endif
```

为了方便，Blade 也提供一个 `@unless` 指令：

```php
@unless (Auth::check())
    You are not signed in.
@endunless
```

除了已经讨论过的条件指令之外，`@isset` 和 `@empty` 指令还可以作为相应 PHP 函数的快捷方式：

```php
@isset($records)
    // $records is defined and is not null...
@endisset

@empty($records)
    // $records is "empty"...
@endempty
```

#### 认证指令

`@auth` 和 `@guest` 指令还可以被用来快速确定当前用户是认证过的或是一个游客：

```php
@auth
    // The user is authenticated...
@endauth

@guest
    // The user is not authenticated...
@endguest
```

如果需要，你可以指定在使用 `@auth` 和 `@guest` 指令时应当检查的 [`认证保护`](https://laravel.com/docs/5.8/authentication)：

```php
@auth('admin')
    // The user is authenticated...
@endauth

@guest('admin')
    // The user is not authenticated...
@endguest
```

#### 部分指令

你可以使用 `@hasSection` 指令检查一个部分是否有内容：

```php
@hasSection('navigation')
    <div class="pull-right">
        @yield('navigation')
    </div>

    <div class="clearfix"></div>
@endif
```

### Switch 语句

可以使用 `@switch`，`@case`，`break`，`@default` 和 `@endswitch` 指令来构造 Switch 语句：

```php
@switch($i)
    @case(1)
        First case...
        @break

    @case(2)
        Second case...
        @break

    @default
        Default case...
@endswitch
```

### 循环

除了条件语句之外，Blade 提供了简单的指令来处理 PHP 的循环结构。同样，这些指令的每个功能都与它们对应的 PHP 相同：

```php
@for ($i = 0; $i < 10; $i++)
    The current value is {{ $i }}
@endfor

@foreach ($users as $user)
    <p>This is user {{ $user->id }}</p>
@endforeach

@forelse ($users as $user)
    <li>{{ $user->name }}</li>
@empty
    <p>No users</p>
@endforelse

@while (true)
    <p>I'm looping forever.</p>
@endwhile
```

{% hint style="info" %}

当循环时，你可以使用 [循环变量](https://laravel.com/docs/5.8/blade#the-loop-variable) 来获取关于循环的有价值的信息，比如你是否在循环的第一次或者最后一次迭代中。

{% endhint %}

当使用循环时，你还可以结束循环或者跳过当前迭代：

```php
@foreach ($users as $user)
    @if ($user->type == 1)
        @continue
    @endif

    <li>{{ $user->name }}</li>

    @if ($user->number == 5)
        @break
    @endif
@endforeach
```

你还可以在一行中包含带有指令的声明条件：

```php
@foreach ($users as $user)
    @continue($user->type == 1)

    <li>{{ $user->name }}</li>

    @break($user->number == 5)
@endforeach
```

### 循环变量

当循环时，一个 `$loop` 变量将在循环内部可以。该变量提供对一些有用信息的访问，比如当前循环索引及是否这是循环的第一次迭代还是最后一次迭代：

```php
@foreach ($users as $user)
    @if ($loop->first)
        This is the first iteration.
    @endif

    @if ($loop->last)
        This is the last iteration.
    @endif

    <p>This is user {{ $user->id }}</p>
@endforeach
```

如果在嵌套循环中，你可以通过循环的 `parent` 属性访问父循环的 `$loop` 变量：

```php
@foreach ($users as $user)
    @foreach ($user->posts as $post)
        @if ($loop->parent->first)
            This is first iteration of the parent loop.
        @endif
    @endforeach
@endforeach
```

`$loop` 变量也包含各种其它用户的属性：

| 属性 | 描述 |
| `$loop->index` | 当前循环迭代的索引（从 0 开始） |
| `$loop->iteration` | 当前循环迭代（从 1 开始） |
| `$loop->remaining` | 循环中剩余的迭代 |
| `$loop->count` | 迭代数组中的项目总数 |
| `$loop->first` | 这是否是循环的第一次迭代 |
| `$loop->last` | 这是否是循环的最后一次迭代 |
| `$loop->even` | 这是否是循环中的偶数迭代 |
| `$loop->odd` | 这是否是循环中的奇数迭代 |
| `$loop->depth` | 当前循环的嵌套级别 |
| `$loop->parent` | 在嵌套循环中，父级的循环变量 |

### 注释

Blade 还允许你在你的视图去定义注释。但是，不像 HTML 注释，Blade 注释不包含在通过应用程序返回的 HTML 中：

```php
{{-- This comment will not be present in the rendered HTML --}}
```

### PHP

在某些情况下，将 PHP 代码嵌入到你的视图中是有用的。我可以使用 Blade `@php` 指令在模版中去执行一个纯 PHP 代码块：

```php
@php
    //
@endphp
```

{% hint style="info" %}

虽然 Blade 提供了这个特性，但频繁使用它可能会意味着模版中嵌入了过多的逻辑。

{% endhint %}

## 表单

### CSRF 字段

无论何时在你的应用程序中定义一个 HTML 表单，你应该在表单中包含一个隐藏的 CSRF 令牌字段以便于 [CSRF 保护](https://laravel.com/docs/5.8/csrf) 中间件能验证请求。你可以使用 `@csrf` Blade 指令去生成令牌字段：

```php
<form method="POST" action="/profile">
    @csrf

    ...
</form>
```

### 方法字段

由于 HTML 表单不能生成 `PUT`，`PATCH` 或 `DELETE` 请求，因此需要添加一个隐藏 `_method` 字段去欺骗这些 HTTP 动词。`@method` Blade 指令可以为你创建：

```php
<form action="/foo/bar" method="POST">
    @method('PUT')

    ...
</form>
```

### 验证错误

`@errors` 指令可以被用来快速检查一个给定的属性是否存在 [验证错误消息](https://laravel.com/docs/5.8/validation#quick-displaying-the-validation-errors)。在 `@errors` 指令中，你可以回显 `$message` 变量去显示错误消息：

```php
<!-- /resources/views/post/create.blade.php -->

<label for="title">Post Title</label>

<input type="text" class="@error('title') is-invalid @enderror">

@error('title')
    <div class="alert alert-danger">{{ $message }}</div>
@enderror
```

## 包含子视图

Blade `@include` 指令允许你从另一个视图中包含一个 Blade 视图。父视图可用的所有变量将对包含的视图可用：

```php
<div>
    @include('shared.errors')

    <form>
        <!-- Form Contents -->
    </form>
</div>
```

即使包含的视图将继承父视图中所有可用的数据，你也可以将一组额外的数组数据传递给包含的视图：

```php
@include('view.name', ['some' => 'data'])
```

如果你企图去 `@include` 个不存在的视图，Laravel 将抛出一个错误。如果你想包含一个可能存在或不存在的视图，你应当使用 `@includeIf` 指令：

```php
@includeIf('view.name', ['some' => 'data'])
```

如果你希望去 `@include` 一个视图根据一个给定的布尔条件，你可以使用 `@includeWhen` 指令：

```php
@includeWhen($boolean, 'view.name', ['some' => 'data'])
```

要包含一个给定的视图数组中存在的第一个视图，可以使用 `includeFirst` 指令：

```php
@includeFirst(['custom.admin', 'admin'], ['some' => 'data'])
```

{% hint style="danger" %}

你应该避免在你的 Blade 视图中使用 `__DIR__` 和 `__FILE__` 常量，

{% endhint %}

**别名包含**

如果你的 Blade 包含存储在一个子目录中，为了容易访问你可以希望去别名它们。例如，假设存储在 `resources/views/includes/input.blade.php` 的一个 Blade 包含以下内容：

```php
<input type="{{ $type ?? 'text' }}">
```

你可以使用 `include` 方法去别名从 `includes.input` 到 `input` 的包含。通常，你应该在 `AppServiceProvider` 类的 `boot` 方法中完成：

```php
use Illuminate\Support\Facades\Blade;

Blade::include('includes.input', 'input');
```

一旦包含被别名，你可以使用别名作为 Blade 指令渲染它：

```php
@input(['type' => 'email'])
```

### 渲染视图集合

你可以使用 Blade 的 `@each` 指令将循环和包含合并成一行：

```php
@each('view.name', $jobs, 'job')
```

第一个参数是数组或集合中每个元素要渲染的局部视图。第二个参数是要迭代的数组或集合，而第三个参数是将分配给视图中当前迭代的变量名。因此，例如，如果你正在迭代一个 `jobs` 数组，通常你将希望在视图部分以一个 `job` 变量的形式访问每个作业。当前迭代的键将作为视图部分中的 `key` 变量可用。

你可以传递第四个参数到 `@each` 指令中。如果给定的数组为空，这个参数决定将渲染的视图。

```php
@each('view.name', $jobs, 'job', 'view.empty')
```

{% hint style="danger" %}

通过 `@each` 渲染的视图不会继承来自父视图中的变量。如果子视图需要这些变量，你应当使用 `@foreach` 和 `@include`。

{% endhint %}

## 堆栈

Blade 允许你推送命名堆栈，这些堆栈可以在另一个视图或布局中的其他地方渲染。这对于指定子视图所需的任何 JavaScript 库特别有用：

```php
@push('scripts')
    <script src="/example.js"></script>
@endpush
```

你可以根据需要多次推送堆栈。要渲染完整的栈内容，将堆栈名称传递到 `@stack` 指令：

```php
<head>
    <!-- Head Contents -->

    @stack('scripts')
</head>
```

如果希望在堆栈的开始部分预先添加内容，你应该使用 `@prepend` 指令：

```php
@push('scripts')
    This will be second...
@endpush

// Later...

@prepend('scripts')
    This will be first...
@endprepend
```

## 服务注入

`@inject` 指令可以用于从 Laravel [服务容器](https://laravel.com/docs/5.8/container) 中检索一个服务。传递到 `@inject` 的第一个参数是要放入的服务的变量名称，而第二个参数你希望解析的服务的类或接口名：

```php
@inject('metrics', 'App\Services\MetricsService')

<div>
    Monthly Revenue: {{ $metrics->monthlyRevenue() }}.
</div>
```

## 扩展 Blade

Blade 允许你使用 `directive` 方法去定义自定义指令。当 Blade 编译器遇到自定义指令时，它将使用该指令包含的表达式调用所提供的回调。

以下示例创建一个 `@datetime($var)` 指令，该指令格式化一个给定的 `$var`，它应该是 `DateTime` 的一个实例：

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Blade;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * 在容器中注册绑定。
     *
     * @return void
     */
    public function register()
    {
        //
    }

    /**
     * 引导任何应用程序服务。
     *
     * @return void
     */
    public function boot()
    {
        Blade::directive('datetime', function ($expression) {
            return "<?php echo ($expression)->format('m/d/Y H:i'); ?>";
        });
    }
}
```

如你所见，我们将 `format` 方法链接到传递给指令的任何表达式上。因此，在这个实例中，这个指令生成的最终 PHP 将是：

```php
<?php echo ($var)->format('m/d/Y H:i'); ?>
```

{% hint style="danger" %}

更新了一个 Blade 指令的逻辑之后，你将需要去删除所有缓存的 Blade 视图。可以使用 `view:clear` Artisan 命令删除缓存的 Blade 视图。

{% endhint %}

### 自定义 If 声明

在定义简单的自定义条件语句时，编写自定义指令有时比必要的要复杂得多。为此，Blade 提供了一个 `Blade::if` 方法，允许你使用闭包快速定义自定义条件指令。例如，让我们定义一个检查当前应用程序环境的自定义条件。我们可以在 `AppServiceProvider` 的 `boot` 方法中这样做：

```php
use Illuminate\Support\Facades\Blade;

/**
 * 引导任何应用程序服务。
 *
 * @return void
 */
public function boot()
{
    Blade::if('env', function ($environment) {
        return app()->environment($environment);
    });
}
```

一旦定义了自定义条件，我们就可以在模板上轻松使用它:

```php
@env('local')
    // The application is in the local environment...
@elseenv('testing')
    // The application is in the testing environment...
@else
    // The application is not in the local or testing environment...
@endenv
```
