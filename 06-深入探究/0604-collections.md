# 集合

## 简介

`Illuminate\Support\Collection` 类为处理数据数组提供了一个流畅、方便的包装器。例如，检查以下代码。我们将使用 `collect` 助手从数组中创建一个新的集合实例，在每个元素上运行 `strtoupper` 函数，然后删除所有空元素：

```php
$collection = collect(['taylor', 'abigail', null])->map(function ($name) {
    return strtoupper($name);
})
->reject(function ($name) {
    return empty($name);
});
```

正如你所看到的，`Collection` 类允许你链接它的方法来执行底层数组的流畅映射和缩减。通常，集合是不可变的，这意味着每个 `Collection` 方法都返回一个全新的 `Collection` 实例。

### 创建集合

如上所述，`collect` 助手为给定数组返回一个新的 `Illuminate\Support\Collection` 实例。因此，创建集合非常简单：

```php
$collection = collect([1, 2, 3]);
```

{% hint style="info" %}

[Eloquent](https://laravel.com/docs/5.8/eloquent) 查询结果集总是作为 `Collection` 实例返回。

{% endhint %}

### 扩展集合

集合是『可宏的』，这允许你在运行时向 `Collection` 类添加其他方法。例如，下面的代码向集合类添加了一个 `toUpper` 方法：

```php
use Illuminate\Support\Str;

Collection::macro('toUpper', function () {
    return $this->map(function ($value) {
        return Str::upper($value);
    });
});

$collection = collect(['first', 'second']);

$upper = $collection->toUpper();

// ['FIRST', 'SECOND']
```

通常，你应该在 [服务提供者](https://laravel.com/docs/5.8/providers) 中声明集合宏。

## 可用的方法

对于本文档的其余部分，我们将讨论 `Collection` 类上可用的每个方法。记住，所有这些方法都可以链接起来流畅地操作底层数组。此外，几乎每个方法都返回一个新的 `Collection` 实例，允许你在必要时保留集合的原始副本：

|                                                                    |                                                                          |                                                                  |
| ------------------------------------------------------------------ | ------------------------------------------------------------------------ | ---------------------------------------------------------------- |
| [all](https://laravel.com/docs/5.8/collections#method-all)         | [isNotEmpty](https://laravel.com/docs/5.8/collections#method-isnotempty) | [sort](https://laravel.com/docs/5.8/collections#method-sort)     |
| [average](https://laravel.com/docs/5.8/collections#method-average) | [join](https://laravel.com/docs/5.8/collections#method-join)             | [sortBy](https://laravel.com/docs/5.8/collections#method-sortby) |

### `all()`

`all` 方法返回通过集合所表示的底层数组：

```php
collect([1, 2, 3])->all();

// [1, 2, 3]
```

### `average()`

`avg` 方法的别名。

### `avg()`

`avg` 方法返回一个给定键的 [平均值](https://en.wikipedia.org/wiki/Average)：

```php
$average = collect([['foo' => 10], ['foo' => 10], ['foo' => 20], ['foo' => 40]])->avg('foo');

// 20

$average = collect([1, 1, 2, 4])->avg();

// 2
```

## 高阶消息
