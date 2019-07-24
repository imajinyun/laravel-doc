# 数据库：分页

## 简介

在其他框架中，分页可能非常痛苦。Laravel 的分页器集成了 [查询生成器](https://laravel.com/docs/5.8/queries) 和 [Eloquent ORM](https://laravel.com/docs/5.8/eloquent)，并提供了方便、易用的开箱即用的数据库结果分页。分页器生成的 HTML 与 [Bootstrap CSS](https://getbootstrap.com/) 框架兼容。

## 基本用法

### 分页查询生成器结果

有几种分页条目的方法。最简单的方法是在 [查询生成器](https://laravel.com/docs/5.8/queries) 或 或者 [Eloquent 查询](https://laravel.com/docs/5.8/eloquent) 上使用 `paginate` 方法。`paginate` 方法自动根据用户查看的当前页面设置适当的限制和偏移量。默认情况下，当前页面由 HTTP 请求上的 `page` 查询字符串参数值检测。Laravel 自动检测此值，并自动将其插入分页器生成的链接中。

在本例中，传递给 `paginate` 方法的惟一参数是你希望『每页』显示的条目数。在本例中，让我们指定你希望在每个页面中显示 `15` 个条目：

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Support\Facades\DB;
use App\Http\Controllers\Controller;

class UserController extends Controller
{
    /**
     * 显示应用程序的所有用户。
     *
     * @return Response
     */
    public function index()
    {
        $users = DB::table('users')->paginate(15);

        return view('user.index', ['users' => $users]);
    }
}
```

{% hint style="danger" %}

目前，通过 Laravel 无法有效地执行使用 `groupBy` 语句的分页操作。如果需要使用带分页结果集的 `groupBy`，建议你查询数据库并手动创建分页器。

{% endhint %}

#### 简单分页

如果你只需要在你的分页视图中显示简单的『下一页』和『上一页』链接，则可以使用 `simplePaginate` 方法执行更有效的查询。当你在渲染你的视图时不需要为每个页码显示链接时，这对于大型数据集非常有用：

```php
$users = DB::table('users')->simplePaginate(15);
```

### 将 Eloquent 结果分页

你还可以分页 [Eloquent](https://laravel.com/docs/5.8/eloquent) 查询。在本例中，我们将用每页 `15` 个条目为 `User` 模型分页。如你所见，语法几乎与分页查询生成器结果相同：

```php
$users = App\User::paginate(15);
```

你可以在设置查询的其他约束（例如：`where` 子句）之后调用 `paginate`：

```php
$users = User::where('votes', '>', 100)->paginate(15);
```

在为 Eloquent 模型分页时，你还可以使用 `simplePaginate` 方法：

```php
$users = User::where('votes', '>', 100)->simplePaginate(15);
```

### 手动创建分页器

有时，你可能希望手动创建分页实例，并将条目数组传递给它。你可以通过创建一个 `Illuminate\Pagination\Paginator` 或 `Illuminate\Pagination\LengthAwarePaginator` 实例来这样做，这取决于你的需要。

`Paginator` 类不需要知道结果集中条目的总数；但是，由于这个原因，该类没有检索最后一页索引的方法。`LengthAwarePaginator` 接受与 `Paginator` 几乎相同的参数；但是，它确实要求对结果集中条目的总数进行计数。

换句话说，`Paginator` 对应于查询生成器和 Eloquent 上的 `simplePaginate` 方法，而 `LengthAwarePaginator` 对应于 `paginate` 方法。

{% hint style="danger" %}

当手动创建分页器实例时，你应该手动『切片』你传递到分页器的结果数组。如果你不确定如何做到这一点，请查看 [array_slice](https://secure.php.net/manual/en/function.array-slice.php) PHP 函数。

{% endhint %}

## 显示分页结果

当调用 `paginate` 方法时，你将接受一个 `Illuminate\Pagination\LengthAwarePaginator` 实例。当调用 `simplePaginate` 方法时，你将接受一个 `Illuminate\Pagination\Paginator` 实例。这些对象提供了几个描述结果集的方法。除了这些助手方法之外，分页器实例是迭代器，并且可以作为数组进行循环。因此，一旦检索到结果，你就可以使用 [Blade](https://laravel.com/docs/5.8/blade) 显示结果并渲染页面链接：

```php
<div class="container">
    @foreach ($users as $user)
        {{ $user->name }}
    @endforeach
</div>

{{ $users->links() }}
```

`links` 方法将渲染结果集中其余页面的链接。每个链接都已经包含了正确的 `page` 查询字符串变量。记住，`links` 方法生成的 HTML 与 [Bootstrap CSS 框架](https://getbootstrap.com/) 兼容。

**自定义分页器 URI**

`withPath` 方法允许你在生成链接时自定义分页器使用的 URI。例如，如果你希望分页器生成 `http://example.com/custom/url?page=N` 等链接，你应该将 `custom/url` 传递给 `withPath` 方法：

```php
Route::get('users', function () {
    $users = App\User::paginate(15);

    $users->withPath('custom/url');

    //
});
```

**附加到分页链接**

你可以使用 `appends` 方法将分页链接的查询字符串追加。例如，要向每个分页链接追加 `sort=vote`，你应该对 `appends` 做以下调用：

```php
{{ $users->appends(['sort' => 'votes'])->links() }}
```

如果你希望去追加一个『哈希片段』到分页器的 URL，你可以使用 `fragment` 方法。例如，要追加 `#foo` 到每个分页器链接的末尾，要对 `fragment` 方法做以下调用：

```php
{{ $users->fragment('foo')->links() }}
```

**调整分页链接窗口**

你可以控制在分页器 URL『窗口』的每一侧显示多少个附加链接。默认情况下，在主分页器链接的每一侧都显示三个链接。但是，你可以使用 `onEachSide` 方法控制这个数字：

```php
{{ $users->onEachSide(5)->links() }}
```

### 将结果转换为 JSON

Laravel 分页器结果类实现了 `Illuminate\Contracts\Support\Jsonable` 接口契约并暴露了 `toJson` 方法，因此很容易将你的分页结果转换为 JSON。你还可以通过从路由或控制器动作返回分页器实例，将其转换为 JSON：

```php
Route::get('users', function () {
    return App\User::paginate();
});
```

来自分页器的 JSON 将包含元信息，如：`total`、`current_page`、`last_page` 等。实际的结果对象将通过 JSON 数组中的 `data` 键可用。下面是一个通过从一个路由返回分页器实例而创建的 JSON 示例：

```php
{
   "total": 50,
   "per_page": 15,
   "current_page": 1,
   "last_page": 4,
   "first_page_url": "http://laravel.app?page=1",
   "last_page_url": "http://laravel.app?page=4",
   "next_page_url": "http://laravel.app?page=2",
   "prev_page_url": null,
   "path": "http://laravel.app",
   "from": 1,
   "to": 15,
   "data":[
        {
            // Result Object
        },
        {
            // Result Object
        }
   ]
}
```

## 自定义分页视图

默认情况下，渲染分页链接的视图与 Bootstrap CSS 框架兼容。但是，如果不使用 Bootstrap，你可以自由定义你自己的视图来渲染这些链接。在分页器实例上调用 `links` 方法时，将视图名作为第一个参数传递给该方法：

```php
{{ $paginator->links('view.name') }}

// 传递数据到视图...
{{ $paginator->links('view.name', ['foo' => 'bar']) }}
```

但是，自定义分页视图的最简单方法是使用 `vendor:publish` 命令将它们导出到 `resources/views/vendor` 目录：

```php
php artisan vendor:publish --tag=laravel-pagination
```

此命令将视图放在 `resources/views/vendor/pagination` 目录中。此目录中的 `bootstrap-4.blade.php` 文件对应于默认的分页视图。你可以编辑此文件以修改分页 HTML。

如果希望将另一个文件指定为默认分页视图，可以在 `AppServiceProvider` 中使用分页器的 `defaultView` 和 `defaultSimpleView` 方法：

```php
use Illuminate\Pagination\Paginator;

public function boot()
{
    Paginator::defaultView('view-name');

    Paginator::defaultSimpleView('view-name');
}
```

## 分页器实例方法

每个分页器实例都通过以下方法提供额外的分页信息：

| 方法                                  | 描述                                                           |
| ------------------------------------- | -------------------------------------------------------------- |
| `$results->count()`                   | 获取当前页面的条目数                                           |
| `$results->currentPage()`             | 获取当前页码                                                   |
| `$results->firstItem()`               | 获取结果中第一编号的条目结果                                   |
| `$results->getOptions()`              | 获取分页器选项                                                 |
| `$results->getUrlRange($start, $end)` | 创建一个带范围的分页 URL                                       |
| `$results->hasMorePages()`            | 确定是否有足够的条目可以拆分为多页                             |
| `$results->items()`                   | 获取当前页面的条目                                             |
| `$results->lastItem()`                | 获取结果中最后编号的条目结果                                   |
| `$results->lastPage()`                | 获取最后一个可用页面的页码（使用 `simplePaginate` 时不可用）   |
| `$results->nextPageUrl()`             | 获取下一页的 URL                                               |
| `$results->onFirstPage()`             | 确定分页器是否在第一页上                                       |
| `$results->perPage()`                 | 每页显示的条目数                                               |
| `$results->previousPageUrl()`         | 获取前一页的 URL                                               |
| `$results->total()`                   | 确定数据存储中匹配条目的总数（使用 `simplePaginate` 时不可用） |
| `$results->url($page)`                | 获取给定页码的 URL                                             |
