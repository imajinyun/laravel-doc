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

## 创建策略

### 生成策略

### 注册策略

## 编写策略

### 策略模型

### 无模型的方法

### Guest 用户

### 策略过滤

## 使用策略授权动作

### 通过用户模型

### 通过中间件

### 通过控制器助手

### 通过 Blade 模版
