# 验证

## 简介

Laravel 提供几个不同的方式去验证你的应用程序传入的数据。默认情况下，Laravel 的基控制器类使用 `ValidatesRequests` 特性，该特性提供一种方便的方法以各种强大的验证规则去验证传入的 HTTP 请求。

## 验证快速入门

要了解关于 Laravel 的强大验证功能，让我们看一个验证表单并将错误消息显示回用户的完整示例。

### 定义路由

首先，让我们假设我们有如下路由定义在我们的 `routes/web.php` 文件：

```php
Route::get('post/create', 'PostController@create');

Route::post('post', 'PostController@store');
```

`GET` 路由将显示用户创建的一个新博客文章的表单，而 `POST` 路由将存储新博客文章到数据库。

### 创建控制器

接下来，我们来看一个处理这些路由的简单控制器。我们现在将 `store` 方法留空：

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use App\Http\Controllers\Controller;

class PostController extends Controller
{
    /**
     * 展示表单去创建一个新博客文章。
     *
     * @return Response
     */
    public function create()
    {
        return view('post.create');
    }

    /**
     * 存储一个新博客文章。
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        // 验证和存储博客文章...
    }
}
```

### 编写验证逻辑

现在我们准备用逻辑来填充我们的 `store` 方法去验证新的博客文章。要这样做，我们将使用 `Illuminate\Http\Request` 对象提供的 `validate` 方法。如果验证规则通过，你的代码将继续正常执行；但是，如果验证失败，将抛出一个异常并且合适的错误响应将自动发送回用户。在这种传统的 HTTP 请求情况下，将生成一个重定向响应，同时为 AJAX 请求发送一个 JSON 响应。

为了更好地理解 `validate` 方法，让我们跳回到 `store` 方法：

```php
/**
 * 存储一个新的博客文章。
 *
 * @param  Request  $request
 * @return Response
 */
public function store(Request $request)
{
    $validatedData = $request->validate([
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
    ]);

    // 博客文章验证通过...
}
```

如你所见，我们传递期望的验证规则到 `validate` 方法。同样，如果验证失败，合适的响应将自动生成。如果验证通过，我们的控制器将继续正常执行。

### 停止首次验证失败

有时我们希望第一次验证失败后对属性停止运行验证规则。为此，分配 `bail` 规则到属性：

```php
$request->validate([
    'title' => 'bail|required|unique:posts|max:255',
    'body' => 'required',
]);
```

在该示例中，如果 `title` 属性的 `unique` 规则验证失败，`max` 规则将不被检查。规则将按它们分配的顺序验证。

#### 关于嵌套属性的注意事项

如果你的 HTTP 请求包含『嵌套』参数，你可以在你的验证规则中使用『点』语法来指定它们：

```php
$request->validate([
    'title' => 'required|unique:posts|max:255',
    'author.name' => 'required',
    'author.description' => 'required',
]);
```

### 显示验证错误

那么，如果传入的请求参数没有通过验证规则该怎么办？如前所述，Laravel 将自动重定向用户回到他们之前的位置。此外，所有的验证错误将自动 [闪存到会话](https://laravel.com/docs/5.8/session#flash-data)。

再次，注意我们不必显式绑定错误消息到 `GET` 路由的视图。这是因为 Laravel 将检查会话数据中的错误，并自动绑定它们到视图（如果可用）。`$errors` 变量是 `Illuminate\Support\MessageBag`的一个实例。有关使用此对象的更多信息，[查看其文档](https://laravel.com/docs/5.8/validation#working-with-error-messages)。

{% hint style="info" %}

`$errors` 变量通过 `Illuminate\View\Middleware\ShareErrorsFromSession` 中间件绑定到视图，它通过 `web` 中间件组提供。**当此中间件被应用时，在你的视图中一个 `$errors` 变量将总是可用**，允许你方便地假定 `$errors` 变量始终定义并可以安全使用。

{% endhint %}

因此，在我们的示例中，当验证失败时，用户将被重定向到我们的控制器的 `create` 方法，允许我们在视图中去显示错误消息：

```php
<!-- /resources/views/post/create.blade.php -->

<h1>创建文章</h1>

@if ($errors->any())
    <div class="alert alert-danger">
        <ul>
            @foreach ($errors->all() as $error)
                <li>{{ $error }}</li>
            @endforeach
        </ul>
    </div>
@endif

<!-- 创建文章表单 -->
```

### 关于可选字段的注意事项

默认情况下，Laravel 在你的应用程序的全局中间件堆栈中包含 `TrimStrings` 和 `ConvertEmptyStringsToNull` 中间件。这些中间件通过 `App\Http\Kernel` 类列在堆栈中。正因如此，如果你不想验证程序将 `null` 值视为无效，则通常需要去标记你的『可选的』请求字段作为 `nullable`。例如：

```php
$request->validate([
    'title' => 'required|unique:posts|max:255',
    'body' => 'required',
    'publish_at' => 'nullable|date',
]);
```

在这个实例中，我们指定 `publish_at` 字段即可为 `null` 也可为有效的日期表示。如果 `nullable` 修饰符没有添加到规则定义中，验证程序将 `null` 视为无效日期。

#### AJAX 请求 & 验证

在此实例中，我们使用传统的表单去发送数据到应用程序。然而，许多应用程序使用 AJAX 请求。当在一个 AJAX 请求期间使用 `validate` 方法，Laravel 将不生成一个重定向响应。相反，Laravel 生成一个包含所有验证错误的 JSON 响应。此 JSON 响应将与一个 422 HTTP 状态码一并发送。

## 表单请求验证

对于更复杂的验证场景，你可能希望去创建一个『表单请求』。表单请求是包含验证逻辑的自定义请求类。要创建一个表单请求类，使用 `make:request` Artisan CLI 命令：

```php
php artisan make:request StoreBlogPost
```

生成的类将放置在 `app/Http/Requests` 目录。如果此目录不存在，当你运行 `make:request` 命令时将创建。让我们添加一些验证规则到 `rules` 方法：

```php
/**
 * 获取应用到请求的验证规则。
 *
 * @return array
 */
public function rules()
{
    return [
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
    ];
}
```

{% hint style="info" %}

你可以在 `rules` 方法的签名上键入类型提示所需的任何依赖项。它们将通过 Laravel [服务容器](https://laravel.com/docs/5.8/container) 自动解析。

{% endhint %}

那么，如何评估验证规则呢？所有你需要做的就是在你的控制器方法上键入类型提示请求。在调用控制器的方法之前验证传入的表单请求，意味着你不需要你使用任何验证逻辑掺杂到你的控制器中：

```php
/**
 * 存储传入的博客文章。
 *
 * @param  StoreBlogPost  $request
 * @return Response
 */
public function store(StoreBlogPost $request)
{
    // 传入的请求通过验证...

    // 检索通过验证的输入数据...
    $validated = $request->validated();
}
```

### 创建表单请求

### 授权表单请求

### 自定义错误消息

### 自定义验证属性

## 手动创建验证器

## 处理错误消息

## 可用的验证规则

## 有条件地添加规则

## 验证数组

## 自定义验证规则
