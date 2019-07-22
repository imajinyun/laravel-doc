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

### 确定是否存在记录

你可以使用 `exists` 和 `doesntExist` 方法，而不是使用 `count` 方法来确定是否存在匹配你的查询约束的记录：

```php
return DB::table('orders')->where('finalized', 1)->exists();

return DB::table('orders')->where('finalized', 1)->doesntExist();
```

## Selects

### 指定 Select 子句

你可能不总是希望从数据库表中选择所有列。使用 `select` 方法，你可以为查询指定一个自定义 `select` 子句：

```php
$users = DB::table('users')->select('name', 'email as user_email')->get();
```

`distinct 方法允许你强制查询返回不同的结果：

```php
$users = DB::table('users')->distinct()->get();
```

如果你已经有一个查询生成器实例，并且你希望向其现有的 Select 子句中添加一个列，你可以使用 `addSelect` 方法：

```php
$query = DB::table('users')->select('name');

$users = $query->addSelect('age')->get();
```

## 原始表达式

有时你可能需要在查询中使用原始表达式。要创建原始表达式，你可以使用 `DB::raw` 方法：

```php
$users = DB::table('users')
                     ->select(DB::raw('count(*) as user_count, status'))
                     ->where('status', '<>', 1)
                     ->groupBy('status')
                     ->get();
```

{% hint style="danger" %}

原始语句将作为字符串注入到查询中，因此你应该非常小心，不要创建 SQL 注入漏洞。

{% endhint %}

### 原始方法

你还可以使用以下方法将原始表达式插入查询的各个部分，而不是使用 `DB::raw`。

#### `selectRaw`

`selectRaw` 方法可以代替 `select(DB::raw(…))`。此方法接受可选的绑定数组作为其第二个参数：

```php
$orders = DB::table('orders')
                ->selectRaw('price * ? as price_with_tax', [1.0825])
                ->get();
```

#### `whereRaw / orWhereRaw`

`whereRaw` 和 `orWhereRaw` 方法可用于将原始 `where` 子句注入到你的查询。这些方法接受一个可选的绑定数组作为它们的第二个参数：

```php
$orders = DB::table('orders')
                ->whereRaw('price > IF(state = "TX", ?, 100)', [200])
                ->get();
```

#### `havingRaw / orHavingRaw`

可以使用 `havingRaw` 和 `orHavingRaw` 方法将原始字符串设置为 `having` 子句的值。这些方法接受一个可选的绑定数组作为它们的第二个参数：

```php
$orders = DB::table('orders')
                ->select('department', DB::raw('SUM(price) as total_sales'))
                ->groupBy('department')
                ->havingRaw('SUM(price) > ?', [2500])
                ->get();
```

#### `orderByRaw`

`orderByRaw` 方法可用于将原始字符串设置为 `order by` 子句的值：

```php
$orders = DB::table('orders')
                ->orderByRaw('updated_at - created_at DESC')
                ->get();
```

## Joins

### Inner Join 子句

查询生成器还可以用于编写连接语句。要执行基本的『inner join』，你可以在查询生成器实例上使用 `join` 方法。传递给 `join` 方法的第一个参数是你需要连接到的表的名称，其余参数指定连接的列约束。你甚至可以在一个查询中连接到多个表：

```php
$users = DB::table('users')
            ->join('contacts', 'users.id', '=', 'contacts.user_id')
            ->join('orders', 'users.id', '=', 'orders.user_id')
            ->select('users.*', 'contacts.phone', 'orders.price')
            ->get();
```

### Left Join / Right Join 子句

如果你希望执行『left join』或者『right join』，而不是『inner join』，请使用 `leftJoin` 或 `rightJoin` 方法。这些方法具有与 `join` 方法相同的签名：

```php
$users = DB::table('users')
            ->leftJoin('posts', 'users.id', '=', 'posts.user_id')
            ->get();

$users = DB::table('users')
            ->rightJoin('posts', 'users.id', '=', 'posts.user_id')
            ->get();
```

### Cross Join 子句

要执行『cross join』，要使用你希望交叉连接的表名的 `crossJoin` 方法。交叉连接在第一个表和连接的表之间生成笛卡尔积：

```php
$users = DB::table('sizes')
            ->crossJoin('colours')
            ->get();
```

### 高级连接子句

你还可以指定更高级的联接子句。首先，将一个 `Closure` 作为第二个参数传递给 `join` 方法。`Closure` 将接收一个 `JoinClause` 对象，该对象允许你在 `join` 子句上指定约束：

```php
DB::table('users')
        ->join('contacts', function ($join) {
            $join->on('users.id', '=', 'contacts.user_id')->orOn(...);
        })
        ->get();
```

如果你希望在你的连接上使用『where』样式子句，你可以在连接上使用 `where` 和 `orWhere` 方法。这些方法不是比较两列，而是将列与值进行比较：

```php
DB::table('users')
        ->join('contacts', function ($join) {
            $join->on('users.id', '=', 'contacts.user_id')
                 ->where('contacts.user_id', '>', 5);
        })
        ->get();
```

### 子查询连接

你可以使用 `joinSub`、`leftJoinSub` 和 `rightJoinSub` 方法将查询连接到一个子查询。每个方法都接收三个参数：子查询、表别名和定义相关列的闭包：

```php
$latestPosts = DB::table('posts')
                   ->select('user_id', DB::raw('MAX(created_at) as last_post_created_at'))
                   ->where('is_published', true)
                   ->groupBy('user_id');

$users = DB::table('users')
        ->joinSub($latestPosts, 'latest_posts', function ($join) {
            $join->on('users.id', '=', 'latest_posts.user_id');
        })->get();
```

## Unions

查询生成器还提供了将两个查询『联合』在一起的快速方法。例如，你可以创建一个初始查询，并使用 `union` 方法将其与第二个查询进行联合：

```php
$first = DB::table('users')
            ->whereNull('first_name');

$users = DB::table('users')
            ->whereNull('last_name')
            ->union($first)
            ->get();
```

{% hint style="info" %}

`unionAll` 方法也可用，并且具有与 `union` 相同的方法签名。

{% endhint %}

## Where 子句

### 简单 Where 子句

你可以在查询生成器实例上使用 `where` 方法向查询添加 `where` 子句。最基本的 `where` 调用需要三个参数。第一个参数是列的名称。第二个参数是操作符，它可以是数据库支持的任何操作符。最后，第三个参数是根据列计算的值。

例如，下面的查询验证『投票』列的值是否等于 100：

```php
$users = DB::table('users')->where('votes', '=', 100)->get();
```

为方便起见，如果你想验证列是否等于给定值，你可以将该值直接作为第二个参数传递给 `where` 方法：

```php
$users = DB::table('users')->where('votes', 100)->get();
```

在编写 `where` 子句时，可以使用多种其他操作符：

```php
$users = DB::table('users')
                ->where('votes', '>=', 100)
                ->get();

$users = DB::table('users')
                ->where('votes', '<>', 100)
                ->get();

$users = DB::table('users')
                ->where('name', 'like', 'T%')
                ->get();
```

你还可以向 `where` 函数传递一个条件数组：

```php
$users = DB::table('users')->where([
    ['status', '=', '1'],
    ['subscribed', '<>', '1'],
])->get();
```

### Or 语句

你可以将约束链接在一起，也可以向查询添加 `or` 子句。`orWhere` 方法接受与 `where` 方法相同的参数：

```php
$users = DB::table('users')
                    ->where('votes', '>', 100)
                    ->orWhere('name', 'John')
                    ->get();
```

### 其它的 Where 子句

#### `whereBetween / orWhereBetween`

#### `whereNotBetween / orWhereNotBetween`

#### `whereIn / whereNotIn / orWhereIn / orWhereNotIn`

#### `whereNull / whereNotNull / orWhereNull / orWhereNotNull`

#### `whereDate / whereMonth / whereDay / whereYear / whereTime`

#### `whereColumn / orWhereColumn`

### 参数分组

## Ordering, Grouping, Limit, & Offset

## 条件子句

## Inserts

## Updates

## Deletes

## 悲观锁

## 调试
