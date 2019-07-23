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

## 显示分页结果

## 自定义分页视图

## 分页器实例方法
