# 数据库：查询生成器

## 简介

Laravel 的数据库查询生成器为创建和运行数据库查询提供了一个方便、流畅的接口。它可以用于在你的应用程序中执行大多数数据库操作，并在所有受支持的数据库系统上工作。

Laravel 查询构建器使用 PDO 参数绑定来保护你的应用程序免受 SQL 注入攻击。不需要清除作为绑定传递的字符串。

{% hint style="danger" %}

PDO 不支持绑定列名。因此，你不应该允许用户指定你的查询引用的列名，包括『order by』列等。如果必须允许用户选择要查询的某些列，始终根据允许列的白名单验证列名。

{% endhint %}

## 检索结果

### 从表中检索所有行

可以使用 `DB` 外观上的 `table` 方法开始查询。`table` 方法为给定的表返回一个连贯的查询生成器实例，允许你将更多约束链到查询上，然后使用 `get` 方法最终获得结果：

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Support\Facades\DB;
use App\Http\Controllers\Controller;

class UserController extends Controller
{
    /**
     * 显示应用程序所有用户的列表。
     *
     * @return Response
     */
    public function index()
    {
        $users = DB::table('users')->get();

        return view('user.index', ['users' => $users]);
    }
}
```

`get` 方法返回一个 `Illuminate\Support\Collection`，该集合包含的结果集中的每个结果都是 PHP `stdClass` 对象的一个实例。你可以通过将列作为对象的属性访问来访问每个列的值：

```php
foreach ($users as $user) {
    echo $user->name;
}
```

### 从表中检索单行 / 列

如果你只需要从数据库表中检索一行，你可以使用 `first` 方法。这个方法将返回一个 `stdClass` 对象：

```php
$user = DB::table('users')->where('name', 'John')->first();

echo $user->name;
```

如果你甚至不需要整行，你可以使用 `value` 方法从记录中提取单个值。这个方法将直接返回列的值：

```php
$email = DB::table('users')->where('name', 'John')->value('email');
```

要根据 `id` 列值检索单个行，要使用 `find` 方法：

```php
$user = DB::table('users')->find(3);
```

### 检索列值列表

如果你希望检索包含单个列值的集合，你可以使用 `pluck` 方法。在本例中，我们将检索角色标题的集合：

```php
$titles = DB::table('roles')->pluck('title');

foreach ($titles as $title) {
    echo $title;
}
```

你还可以为返回的集合指定自定义键列：

```php
$roles = DB::table('roles')->pluck('title', 'name');

foreach ($roles as $name => $title) {
    echo $title;
}
```

### 分块结果集

如果你需要处理数千条数据库记录，考虑使用 `chunk` 方法。此方法每次检索一小块结果，并将每个块提供给一个 `Closure` 进行处理。这种方法对于编写处理数千条记录的 [Artisan 命令](https://laravel.com/docs/5.8/artisan) 非常有用。例如，让我们将整个 `users` 表的记录分割成一次处理 100 条记录的小块：

```php
DB::table('users')->orderBy('id')->chunk(100, function ($users) {
    foreach ($users as $user) {
        //
    }
});
```

你可以通过从 `Closure` 返回 `false` 来阻止进一步的块被处理：

```php
DB::table('users')->orderBy('id')->chunk(100, function ($users) {
    // 处理记录...

    return false;
});
```

如果在对结果进行分块时更新数据库记录，你的块结果可能会以意想不到的方式更改。因此，在分块时更新记录时，最好使用 `chunkById` 方法。该方法将根据记录的主键自动分页结果：

```php
DB::table('users')->where('active', false)
    ->chunkById(100, function ($users) {
        foreach ($users as $user) {
            DB::table('users')
                ->where('id', $user->id)
                ->update(['active' => true]);
        }
    });
```

{% hint style="danger" %}

当更新或删除块回调中的记录时，对主键或外键的任何更改都可能影响块查询。这可能导致记录没有包含在分块结果中。

{% endhint %}

### 聚合

查询生成器还提供了各种聚合方法，如：`count`、`max`、`min`、`avg` 和 `sum`。你可以在构造查询之后调用这些方法中的任何一个：

```php
$users = DB::table('users')->count();

$price = DB::table('orders')->max('price');
```

你可以将这些方法与其他子句结合使用：

```php
$price = DB::table('orders')
                ->where('finalized', 1)
                ->avg('price');
```

## Selects

## 原始表达式

## Joins

## Unions

## Where 子句

## Ordering, Grouping, Limit, & Offset

## 条件子句

## Inserts

## Updates

## Deletes

## 悲观锁

## 调试
