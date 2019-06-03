# 授权

## 简介

除了提供开箱即用的 [认证](https://laravel.com/docs/5.8/authentication) 服务之外，Laravel 还提供了一种简单的方法来授权用户对给定资源的操作。与认证一样，Laravel 的授权方法也很简单，有两种主要的授权操作方法：gates 和策略。

想想 Gates 和策略，比如路由和控制器。Gates 提供了一种简单的、基于闭包的授权方法，而策略（如控制器）则根据特定的模型或资源对其逻辑进行分组。我们将首先探究 Gates，然后探究策略。

在构建应用程序时，你不需要在专用 Gates 和专用策略之间进行选择。大多数应用程序很可能包含 Gates 和策略的混合，这非常好！Gates 最适用于与任何模型或资源无关的动作，例如：查看管理员仪表板。相反，当你希望授权特定模型或资源的动作时，应该使用策略。

## Gates

### 编写 Gates

Gates 是决定用户是否被授权执行给定动作的闭包，通常使用的 `Gate` facade 的在 `App\Providers\AuthServiceProvider` 类中定义。Gates 总是接收用户实例作为其第一个参数，并且可以选择接收其他参数，比如相关的 Eloquent 模型：

```php
/**
 * 注册任何认证 / 授权服务。
 *
 * @return void
 */
public function boot()
{
    $this->registerPolicies();

    Gate::define('update-post', function ($user, $post) {
        return $user->id == $post->user_id;
    });
}
```

Gates 还可以使用一个 `Class@method` 样式的回调字符串定义，像控制器：

```php
/**
 * 注册任何认证 / 授权服务。
 *
 * @return void
 */
public function boot()
{
    $this->registerPolicies();

    Gate::define('update-post', 'App\Policies\PostPolicy@update');
}
```

### 授权动作

要授权使用 Gates 的动作，你应该使用 `allows` 或 `denies` 方法。注意，不需要将当前经过认证的用户传递给这些方法。Laravel 自动将用户传递到 Gate 的 Closure：

```php
if (Gate::allows('update-post', $post)) {
    // 当前用户能可以新文章...
}

if (Gate::denies('update-post', $post)) {
    // 当前用户不能更新文章...
}
```

如果你想确定某个特定用户是否被授权执行某个动作，可以在 `Gate` facade 上使用 `forUser` 方法：

```php
if (Gate::forUser($user)->allows('update-post', $post)) {
    // 当前用户能可以新文章...
}

if (Gate::forUser($user)->denies('update-post', $post)) {
    // 当前用户不能更新文章...
}
```

### Gate 拦截检查

有时，你可能希望将所有能力授予特定的用户。你可以使用 `before` 方法来定义一个在所有其他授权检查之前运行的回调：

```php
Gate::before(function ($user, $ability) {
    if ($user->isSuperAdmin()) {
        return true;
    }
});
```

如果 `before` 回调返回非空结果，则该结果将被视为检查的结果。

你可以使用 `after` 方法去定义一个在所有其它授权检查之后执行的回调：

```php
Gate::after(function ($user, $ability, $result, $arguments) {
    if ($user->isSuperAdmin()) {
        return true;
    }
});
```

类似于 `before` 检查，如果 `after` 回调所有一个非空结果，则该结果将被视为检查的结果。

## 创建策略

### 生成策略

策略是围绕特定模型或资源组织授权逻辑的类。例如，如果你的应用程序是一个博客，你可能有一个 `Post 模型和相应的 `PostPolicy` 来授权用户动作（比如创建或更新帖子）。

你可以使用 `make:policy` [Artisan 命令](https://laravel.com/docs/5.8/artisan) 生成一个策略。生成的策略位于 `app/Policies` 目录中。如果这个目录在你的应用程序中不存在，Laravel 将为你创建它：

```bash
php artisan make:policy PostPolicy
```

`make:policy` 命令将生成一个空策略类。如果你希望生成一个类，该类中已经包含基本的『CRUD』策略方法，你可以在执行命令时指定一个 `--model` 参数：

```bash
php artisan make:policy PostPolicy --model=Post
```

{% hint style="info" %}

所有的策略通过 Laravel [服务容器](https://laravel.com/docs/5.8/container) 去解析，允许你在策略构造中键入类型提示任何需要的依赖，它们将被自动注入。

{% endhint %}

### 注册策略

一旦策略存在，就需要注册它。新 Laravel 应用程序中包含的 `AuthServiceProvider` 包含一个 `policy` 属性，它将你的 Eloquent 模型映射到它们相应的策略上。注册策略将指示 Laravel 在授权针对给定模型的动作时利用哪个策略：

```php
<?php

namespace App\Providers;

use App\Post;
use App\Policies\PostPolicy;
use Illuminate\Support\Facades\Gate;
use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;

class AuthServiceProvider extends ServiceProvider
{
    /**
     * 应用程序的策略映射。
     *
     * @var array
     */
    protected $policies = [
        Post::class => PostPolicy::class,
    ];

    /**
     * 注册任何应用程序认证 / 授权服务。
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        //
    }
}
```

#### 策略自动发现

只要模型和策略遵循标准的 Laravel 命名约定，Laravel 就可以自动发现策略，而不是手动注册模型策略。具体而言，策略必须位于包含模型的目录下的 `Policies` 目录中。因此，例如，模型可以放在 `app` 目录中，而策略可以放在 `app/Policies` 目录中。此外，策略名称必须与模型名称匹配，并具有策略后缀。因此，`User` 模型将对应于 `UserPolicy` 类。

如果你想提供你自己的策略发现逻辑，你可以使用 `Gate::guessPolicyNamesUsing` 方法去注册一个自定义回调。通常，这个方法应该从你的应用程序的 `AuthServiceProvider` 的 `boot` 方法中被调用：

```php
use Illuminate\Support\Facades\Gate;

Gate::guessPolicyNamesUsing(function ($modelClass) {
    // 返回策略类名称...
});
```

{% hint style="danger" %}

在 `AuthServiceProvider` 中显式映射的任何策略都将优先于任何潜在的自动发现策略。

{% endhint %}

## 编写策略

### 策略方法

一旦注册了策略，你可以为它授权的每个动作添加方法。例如，让我们在 `PostPolicy` 上定义一个 `update` 方法，它确定给定 `User` 是否可以更新给定的 `Post` 实例。

`update` 方法将接收一个 `User` 和一个 `Post` 实例作为它的参数，并且应该返回 `true` 或 `false`，指示用户是否被授权更新给定的 `Post`。因此，在本示例中，让我们验证用户的 `id` 是否与文章上的 `user_id` 匹配：

```php
<?php

namespace App\Policies;

use App\User;
use App\Post;

class PostPolicy
{
    /**
     * 确定用户是否可以更新给定的文章。
     *
     * @param  \App\User  $user
     * @param  \App\Post  $post
     * @return bool
     */
    public function update(User $user, Post $post)
    {
        return $user->id === $post->user_id;
    }
}
```

你可以根据授权的各种动作的需要，继续在策略上定义其他方法。例如，你可以定义 `view` 或 `delete` 方法来授权各种 `Post` 操作，但是请记住，你可以自由地为策略方法指定任何你喜欢的名称。

{% hint style="info" %}

如果你通过 Artisan 控制台生成你的策略时使用 `--model` 选项，它将包含 `view`，`create`，`update`，`delete`，`restore` 和 `forceDelete` 动作方法。

{% endhint %}

### 无模型的方法

一些策略方法只接收当前经过认证的用户，而不接收它们授权的模型的实例。这种情况在授权 `create` 操作时最为常见。例如，如果你正在创建博客，你可能希望检查用户是否被授权创建任何文章。

当定义不接收模型实例的策略方法时，例如一个 `create` 方法，它将不接收模型实例。相反，你应该将方法定义为只期望经过认证的用户：

```php
/**
 * 确定给定用户是否可以创建文章。
 *
 * @param  \App\User  $user
 * @return bool
 */
public function create(User $user)
{
    //
}
```

### Guest 用户

默认情况下，如果传入的 HTTP 请求不是由经过认证的用户发起的，所有的 Gates 和策略都会自动返回 `false`。但是，你可以通过声明『可选』类型提示或为用户参数定义提供 `null` 默认值，从而允许这些授权检查传递到你的 Gates 和策略：

```php
<?php

namespace App\Policies;

use App\User;
use App\Post;

class PostPolicy
{
    /**
     * 确定用户是否可以更新给定的文章。
     *
     * @param  \App\User  $user
     * @param  \App\Post  $post
     * @return bool
     */
    public function update(?User $user, Post $post)
    {
        return $user->id === $post->user_id;
    }
}
```

### 策略过滤

对于某些用户，你可能希望授权给定策略中的所有动作。为此，在策略上定义一个 `before` 方法。`before` 方法将在策略上的任何其他方法之前执行，从而使你有机会在实际调用预期的策略方法之前授权动作。此功能最常用来授权给应用程序的管理员来执行任何动作：

```php
public function before($user, $ability)
{
    if ($user->isSuperAdmin()) {
        return true;
    }
}
```

如果你想拒绝用户的所有授权，你应该从 `before` 方法返回 `false`。如果返回 `null`，则授权将传递给策略方法。

{% hint style="danger" %}

如果策略类不包含具有与正在检查的功能名称匹配的一个名称的方法，则不会调用策略类的 `before` 方法。

{% endhint %}

## 使用策略授权动作

### 通过用户模型

Laravel 应用程序中包含的 `User` 模型包含两个用于授权动作的有用方法：`can` 和 `cant`。`can` 方法接收你希望授权的动作和相关模型。例如，让我们来确定是否授权用户更新给定的 `Post` 模型：

```php
if ($user->can('update', $post)) {
    //
}
```

如果为给定模型 [注册了策略](https://laravel.com/docs/5.8/authorization#registering-policies)，`can` 方法将自动调用适当的策略并返回布尔结果。如果没有为模型注册策略，`can` 方法将尝试调用基于闭包的 Gate，该 Gate 与给定的操作名称匹配。

#### 不需要模型的动作

记住，像 `create` 这样的一些动作可能不需要模型实例。在这些情况下，可以将类名传递给 `can` 方法。类名将用于确定在授权动作时使用哪个策略：

```php
use App\Post;

if ($user->can('create', Post::class)) {
    // Executes the "create" method on the relevant policy...
    // 在相关策略上执行『create』方法...
}
```

### 通过中间件

Laravel 包含一个中间件，可以在传入的请求到达路由或控制器之前对动作进行授权。默认情况下，在应用程序的 `App\Http\Kernel` 类中分配了 `Illuminate\Auth\Middleware\Authorize` 中间件的 `can` 键名。让我们探究一个使用 `can` 中间件授权用户更新博客文章的示例：

```php
use App\Post;

Route::put('/post/{post}', function (Post $post) {
    // 当前用户可以更新文章...
})->middleware('can:update,post');
```

在本例中，我们传递 `can` 中间件两个参数。第一个是希望授权的动作的名称，第二个是希望传递给策略方法的路由参数。在这种情况下，由于我们使用 [隐式模型绑定](https://laravel.com/docs/5.8/routing#implicit-binding)，`Post` 模型将传递给策略方法。如果用户没有被授权执行给定的动作，则中间件将生成一个带有 `403` 状态代码的 HTTP 响应。

#### 不需要模型的动作

同样，像 `create` 这样的一些动作可能不需要模型实例。在这种情况下，可以将类名传递给中间件。类名将用于确定在授权动作时使用哪个策略：

```php
Route::post('/post', function () {
    // 当前用户可以创建文章...
})->middleware('can:create,App\Post');
```

### 通过控制器助手

除了向 `User` 模型提供有用的方法之外，Laravel 还为扩展 `App\Http\Controllers\Controller` 基类的任何控制器提供了一个有用的 `authorize` 方法。与 `can` 方法一样，此方法接受你希望授权的动作的名称和相关模型。如果操作未被授权，`authorize` 方法将抛出一个 `Illuminate\Auth\Access\AuthorizationException` 异常，默认的 Laravel 异常处理程序将使用 `403` 状态代码将其转换为 HTTP 响应：

```php
<?php

namespace App\Http\Controllers;

use App\Post;
use Illuminate\Http\Request;
use App\Http\Controllers\Controller;

class PostController extends Controller
{
    /**
     * 更新给定的博客文章.
     *
     * @param  Request  $request
     * @param  Post  $post
     * @return Response
     * @throws \Illuminate\Auth\Access\AuthorizationException
     */
    public function update(Request $request, Post $post)
    {
        $this->authorize('update', $post);

        // 当前用户可以更新博客文章...
    }
}
```

#### 不需要模型的动作

如前所述，`create` 之类的一些动作可能不需要模型实例。在这种情况下，可以将类名传递给 `authorize` 方法。类名将用于确定在授权动作时使用哪个策略：

```php
/**
 * 创建一个新的博客文章。
 *
 * @param  Request  $request
 * @return Response
 * @throws \Illuminate\Auth\Access\AuthorizationException
 */
public function create(Request $request)
{
    $this->authorize('create', Post::class);

    // 当前用户可以更新博客文章...
}
```

#### 授权资源控制器

### 通过 Blade 模版
