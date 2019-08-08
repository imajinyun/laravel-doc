# Eloquent：集合

## 简介

通过 Eloquent 返回的所有多结果集都是 `Illuminate\Database\Eloquent\Collection` 对象的实例，包括通过 `get` 方法检索的结果或通过关系访问的结果。Eloquent 集合对象继承了 Laravel [基本集合](https://laravel.com/docs/5.8/collections)，因此它自然地继承了许多用于流畅地使用 Eloquent 模型的底层数组的方法。

所有集合也充当迭代器，允许你循环遍历它们，就像它们是简单的 PHP 数组一样：

```php
$users = App\User::where('active', 1)->get();

foreach ($users as $user) {
    echo $user->name;
}
```

但是，集合比数组更强大，并且可以使用直观的界面暴露各种可以链接的 map / reduce 操作。例如，让我们删除所有非活动模型并收集每个遗留用户的名字：

```php
$users = App\User::all();

$names = $users->reject(function ($user) {
    return $user->active === false;
})
->map(function ($user) {
    return $user->name;
});
```

{% hint style="danger" %}

虽然大多数 Eloquent 集合方法返回 Eloquent 集合的新实例，但 `pluck`、`keys`、`zip`、`collapse`、`flatten` 和 `flip` 方法返回一个 [基本集合](https://laravel.com/docs/5.8/collections) 实例。同样，如果 `map` 操作返回一个不包含任何 Eloquent 模型的集合，它将自动转换为基本集合。

{% endhint %}

## 可用方法
