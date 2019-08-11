# Eloquent：API 资源

## 简介

构建 API 时，你可能需要一个位于你的 Eloquent 模型和实际返回给应用程序用户的 JSON 响应之间的转换层。Laravel 的资源类允许你富于表达力且轻松地将你的模型和模型集合转换为 JSON。

## 生成资源

要生成资源类，可以使用 `make:resource` Artisan 命令。默认情况下，资源将放在你的应用程序的 `app/Http/Resources` 目录中。资源继承了 `Illuminate\Http\Resources\Json\JsonResource` 类：

```bash
php artisan make:resource User
```

***

**资源集合**

除了生成转换单个模型的资源之外，你还可以生成负责转换模型集合的资源。这允许你的响应包括与给定资源的整个集合相关的链接和其他元信息。

要创建资源集合，你应当在创建资源时使用 `--collection` 标志。或者，在资源名称中包含单词 `Collection` 将指示 Laravel 它应该创建一个集合资源。集合资源继承了 `Illuminate\Http\Resources\Json\ResourceCollection` 类：

```bash
php artisan make:resource Users --collection

php artisan make:resource UserCollection
```

## 概念概述

{% hint style="info" %}

这是对资源和资源集合的高级概述。我们强烈建议你阅读本文档的其他部分，以深入了解资源为你提供的自定义和功能。

{% endhint %}

在深入研究你编写资源时可用的所有选项之前，让我们先从高层次上了解一下 Laravel 中如何使用资源。资源类表示需要转换为 JSON 结构的单个模型。例如，这里有一个简单的 `User` 资源类：

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Resources\Json\JsonResource;

class User extends JsonResource
{
    /**
     * 将资源转换为一个数组。
     *
     * @param  \Illuminate\Http\Request  $request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }
}
```

每个资源类都定义一个 `toArray` 方法，该方法返回在发送响应时应转换为 JSON 的属性数组。注意，我们可以直接从 `$this` 变量访问模型属性。这是因为资源类自动将属性和方法访问代理到底层模型以便于访问。一旦定义了资源，就可以从路由或控制器返回：

```php
use App\User;
use App\Http\Resources\User as UserResource;

Route::get('/user', function () {
    return new UserResource(User::find(1));
});
```

### 资源集合

如果要返回资源集合或分页响应，你可以在你的路由或控制器中创建资源实例时使用 `collection` 方法：

```php
use App\User;
use App\Http\Resources\User as UserResource;

Route::get('/user', function () {
    return UserResource::collection(User::all());
});
```

注意，这不允许添加任何可能需要与集合一起返回的元数据。如果你希望自定义资源集合响应，你可以创建一个专用的资源来表示集合：

```bash
php artisan make:resource UserCollection
```

一旦生成了资源集合类，你就可以轻松地定义应该包含在响应中的任何元数据：

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Resources\Json\ResourceCollection;

class UserCollection extends ResourceCollection
{
    /**
     * 将资源集合转换为数组。
     *
     * @param  \Illuminate\Http\Request  $request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'data' => $this->collection,
            'links' => [
                'self' => 'link-value',
            ],
        ];
    }
}
```

***

定义了你的资源集合后，它可以从路由或控制器返回：

```php
use App\User;
use App\Http\Resources\UserCollection;

Route::get('/users', function () {
    return new UserCollection(User::all());
});
```

***

**保留集合键**

从路由返回资源集合时，Laravel 会重置集合的键，以便它们按照简单的数字顺序排列。但是，你可以向你的资源类添加 `preserveKeys` 属性，指示是否应保留集合键：

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Resources\Json\JsonResource;

class User extends JsonResource
{
    /**
     * 指示是否应保留资源的集合键。
     *
     * @var bool
     */
    public $preserveKeys = true;
}
```

***

当 `preserveKeys` 属性设置为 `true` 时，集合键将被保留：

```php
use App\User;
use App\Http\Resources\User as UserResource;

Route::get('/user', function () {
    return UserResource::collection(User::all()->keyBy->id);
});
```

***

**自定义基础资源类**

通常，资源集合的 `$this->collection` 属性会自动填充，其结果是将集合的每个条目映射到其单一资源类。假设单一资源类是集合的类名，后面没有 `Collection` 字符串。

例如，`UserCollection` 将尝试将给定的用户实例映射到 `User` 资源。要自定义此行为，你可以覆盖你的资源集合的 `$collect` 属性：

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Resources\Json\ResourceCollection;

class UserCollection extends ResourceCollection
{
    /**
     * 此资源收集的资源。
     *
     * @var string
     */
    public $collects = 'App\Http\Resources\Member';
}
```

***

## 编写资源

{% hint style="info" %}

如果你还没有阅读 [概念概述](https://laravel.com/docs/5.8/eloquent-resources#concept-overview)，强烈建议你在继续阅读本文档之前去阅读。

{% endhint %}

本质上，资源是简单的。它们只需要将给定的模型转换为一个数组。因此，每个资源都包含一个 `toArray` 方法，该方法将模型的属性转换为可返回给你的用户的 API 友好数组：

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Resources\Json\JsonResource;

class User extends JsonResource
{
    /**
     * 资源转换为数组。
     *
     * @param  \Illuminate\Http\Request  $request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }
}
```

***

一旦定义了资源，你就可以直接从路由或控制器返回：

```php
use App\User;
use App\Http\Resources\User as UserResource;

Route::get('/user', function () {
    return new UserResource(User::find(1));
});
```

***

**关系**

如果你希望在你的响应中包含相关资源，你可以将它们添加通过你的 `toArray` 方法返回的数组中。在本例中，我们将使用 `Post` 资源的 `collection` 方法将用户的博客文章添加到资源响应中：

```php
/**
 * 资源转换为数组。
 *
 * @param  \Illuminate\Http\Request  $request
 * @return array
 */
public function toArray($request)
{
    return [
        'id' => $this->id,
        'name' => $this->name,
        'email' => $this->email,
        'posts' => PostResource::collection($this->posts),
        'created_at' => $this->created_at,
        'updated_at' => $this->updated_at,
    ];
}
```

***

{% hint style="info" %}

如果你希望仅在已经加载关系时包含关系，查看有关 [条件关系](https://laravel.com/docs/5.8/eloquent-resources#conditional-relationships) 的文档。

{% endhint %}

**资源集合**

资源将单个模型转换为数组，而资源集合将模型集合转换为数组。并非绝对有必要为你的每个模型类型定义资源集合类，因为所有资源都提供了一个 `collection` 方法来在运行时生成『临时』资源集合：

```php
use App\User;
use App\Http\Resources\User as UserResource;

Route::get('/user', function () {
    return UserResource::collection(User::all());
});
```

***

但是，如果需要自定义随集合返回的元数据，则需要定义资源集合：

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Resources\Json\ResourceCollection;

class UserCollection extends ResourceCollection
{
    /**
     * 资源集合转换为数组。
     *
     * @param  \Illuminate\Http\Request  $request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'data' => $this->collection,
            'links' => [
                'self' => 'link-value',
            ],
        ];
    }
}
```

***

与单一资源一样，资源集合可以直接从路由或控制器返回：

```php
use App\User;
use App\Http\Resources\UserCollection;

Route::get('/users', function () {
    return new UserCollection(User::all());
});
```

***

### 数据包装

默认情况下，当资源响应转换为 JSON 时，你的最外层的资源被包装在一个 `data` 键中。因此，例如，典型的资源收集响应看起来如下所示：

```json
{
    "data": [
        {
            "id": 1,
            "name": "Eladio Schroeder Sr.",
            "email": "therese28@example.com",
        },
        {
            "id": 2,
            "name": "Liliana Mayert",
            "email": "evandervort@example.com",
        }
    ]
}
```

***

如果你要禁用最外层资源的包装，你可以在基本资源类上使用 `withoutWrapping` 方法。通常，你应该从 `AppServiceProvider` 或在你的应用程序的每个请求上加载的其他 [服务提供者](https://laravel.com/docs/5.8/providers) 中调用此方法：

```php
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use Illuminate\Http\Resources\Json\Resource;

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
        Resource::withoutWrapping();
    }
}
```

***

{% hint style="danger" %}

`withoutWrapping` 方法仅影响最外层响应，并且不会删除你手动添加到你自己的资源集合中的 `data` 键。

{% endhint %}

### 包装嵌套资源

你可以完全自由地确定你的资源的关系如何被包装。如果你希望将所有资源集合包装在 `data` 键中，而不管它们如何嵌套，你应为每个资源定义资源集合类，并在 `data` 键中返回该集合。

你可能担心这是否会导致你的最外层资源被包装在两个数据键中。别担心，Laravel 永远不会让你的资源意外地被双重包装，因此你不必担心正在转换的资源集合的嵌套级别：

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Resources\Json\ResourceCollection;

class CommentsCollection extends ResourceCollection
{
    /**
     * 资源集合转换为数组。
     *
     * @param  \Illuminate\Http\Request  $request
     * @return array
     */
    public function toArray($request)
    {
        return ['data' => $this->collection];
    }
}
```

***

### 数据包装并分页

在资源响应中返回分页集合时，即使已调用 `withoutWrapping` 方法，Laravel 也会将你的资源数据包装在 `data` 键中。这是因为分页响应始终包含 `meta` 和 `links` 键以及有关分页器状态的信息：

```json
{
    "data": [
        {
            "id": 1,
            "name": "Eladio Schroeder Sr.",
            "email": "therese28@example.com",
        },
        {
            "id": 2,
            "name": "Liliana Mayert",
            "email": "evandervort@example.com",
        }
    ],
    "links":{
        "first": "http://example.com/pagination?page=1",
        "last": "http://example.com/pagination?page=1",
        "prev": null,
        "next": null
    },
    "meta":{
        "current_page": 1,
        "from": 1,
        "last_page": 1,
        "path": "http://example.com/pagination",
        "per_page": 15,
        "to": 10,
        "total": 10
    }
}
```

***

### 分页

你可以始终将分页器实例传递给资源的 `collection` 方法或自定义资源集合：

```php
use App\User;
use App\Http\Resources\UserCollection;

Route::get('/users', function () {
    return new UserCollection(User::paginate());
});
```

***

分页响应总是包含 `meta` 和 `links` 键，该键是关于分页器状态的信息：

```php
{
    "data": [
        {
            "id": 1,
            "name": "Eladio Schroeder Sr.",
            "email": "therese28@example.com",
        },
        {
            "id": 2,
            "name": "Liliana Mayert",
            "email": "evandervort@example.com",
        }
    ],
    "links":{
        "first": "http://example.com/pagination?page=1",
        "last": "http://example.com/pagination?page=1",
        "prev": null,
        "next": null
    },
    "meta":{
        "current_page": 1,
        "from": 1,
        "last_page": 1,
        "path": "http://example.com/pagination",
        "per_page": 15,
        "to": 10,
        "total": 10
    }
}
```

***

### 条件属性

有时，如果满足给定条件，你可能希望仅在资源响应中包含属性。例如，如果当前用户是『管理员』，你可能希望仅包含值。Laravel 提供了各种辅助方法来帮助你解决这种情况。`when` 方法可用于有条件地将属性添加到一个资源响应：

```php
/**
 * 资源转换为数组。
 *
 * @param  \Illuminate\Http\Request  $request
 * @return array
 */
public function toArray($request)
{
    return [
        'id' => $this->id,
        'name' => $this->name,
        'email' => $this->email,
        'secret' => $this->when(Auth::user()->isAdmin(), 'secret-value'),
        'created_at' => $this->created_at,
        'updated_at' => $this->updated_at,
    ];
}
```

***

在此示例中，如果经过认证的用户的 `isAdmin` 方法返回 `true`，则仅在最终资源响应中返回 `secret`。如果该方法返回 `false`，则在将资源响应发送回客户端之前，将完全从资源响应中删除该 `secret`。`when` 方法允许你在构建数组时富于表达力地定义你的资源，而无需求助于条件语句。

`when` 方法还接受闭包作为其第二个参数，允许你仅在给定条件为 `true` 时计算结果值：

```php
'secret' => $this->when(Auth::user()->isAdmin(), function () {
    return 'secret-value';
}),
```

***

**合并条件属性**

有时，你可能有几个属性，这些属性应该只包含在基于相同条件的资源响应中。在这种情况下，只有在给定条件为 `true` 时，你才可以使用 `mergeWhen` 方法在响应中包含属性：

```php
/**
 * 资源转换为数组。
 *
 * @param  \Illuminate\Http\Request  $request
 * @return array
 */
public function toArray($request)
{
    return [
        'id' => $this->id,
        'name' => $this->name,
        'email' => $this->email,
        $this->mergeWhen(Auth::user()->isAdmin(), [
            'first-secret' => 'value',
            'second-secret' => 'value',
        ]),
        'created_at' => $this->created_at,
        'updated_at' => $this->updated_at,
    ];
}
```

***

同样，如果给定的条件为 `false`，这些属性将在资源响应发送到客户机之前从资源响应中完全删除。

{% hint style="danger" %}

`mergeWhen` 方法不应在混合字符串和数字键的数组中使用。此外，它不应该使用在没有排好序的数字键的数组中。

{% endhint %}

### 条件关系

除了有条件地加载属性之外，你还可以根据模型上是否已加载关系，有条件地在资源响应中包含关系。这允许你的控制器决定应该在模型上加载哪些关系，并且你的资源只有在实际加载时才能轻松包含它们。

最终，这可以更轻松地避免在你的资源中的『N+1』查询问题。`whenLoaded` 方法可用于有条件地加载关系。为了避免不必要地加载关系，此方法接受关系的名称而不是关系本身：

```php
/**
 * 资源转换为数组。
 *
 * @param  \Illuminate\Http\Request  $request
 * @return array
 */
public function toArray($request)
{
    return [
        'id' => $this->id,
        'name' => $this->name,
        'email' => $this->email,
        'posts' => PostResource::collection($this->whenLoaded('posts')),
        'created_at' => $this->created_at,
        'updated_at' => $this->updated_at,
    ];
}
```

***

在本例中，如果没有加载关系，则在将 `posts` 键发送到客户机之前，将从资源响应中完全删除它。

**条件中转信息**

除了条件在你的资源响应中包含关系信息之外，你还可以使用 `whenPivotLoaded` 方法有条件地包含来自多对多关系的中转表的数据。`whenPivotLoaded` 方法接受中转表的名称作为其第一个参数。第二个参数应该是一个闭包，它定义了如果模型上的中转信息可用时返回的值：

```php
/**
 * 资源转换为数组。
 *
 * @param  \Illuminate\Http\Request  $request
 * @return array
 */
public function toArray($request)
{
    return [
        'id' => $this->id,
        'name' => $this->name,
        'expires_at' => $this->whenPivotLoaded('role_user', function () {
            return $this->pivot->expires_at;
        }),
    ];
}
```

***

如果你的中间表使用的是 `pivot` 以外的访问器，你可以使用 `whenPivotLoadedAs` 方法：

```php
/**
 * 资源转换为数组。
 *
 * @param  \Illuminate\Http\Request  $request
 * @return array
 */
public function toArray($request)
{
    return [
        'id' => $this->id,
        'name' => $this->name,
        'expires_at' => $this->whenPivotLoadedAs('subscription', 'role_user', function () {
            return $this->subscription->expires_at;
        }),
    ];
}
```

### 添加元数据

一些 JSON API 标准要求向资源和资源集合响应添加元数据。这通常包括一些像资源或相关资源的 `links`，或关于资源本身的元数据。如果你需要返回关于资源的其他元数据，将其包含在你的 `toArray` 方法中。例如，在转换资源集合时你可能包含 `link` 信息：

```php
/**
 * 资源转换为数组。
 *
 * @param  \Illuminate\Http\Request  $request
 * @return array
 */
public function toArray($request)
{
    return [
        'data' => $this->collection,
        'links' => [
            'self' => 'link-value',
        ],
    ];
}
```

***

从你的资源返回其他元数据时，你永远不必担心意外覆盖 `links` 或 `meta` 键，当返回分页的响应时它们将通过 Laravel 自动添加。你定义的任何其他 `links` 将与分页器提供的链接合并。

**顶级元数据**

有时，如果资源是返回的最外层资源，你可能希望仅将某些元数据包含在资源响应中。通常，这包括关于整个响应的元信息。要定义此元数据，在资源类中添加 `with` 方法。仅当资源是渲染的最外层资源时，此方法才应返回要包含在资源响应中的元数据数组：

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Resources\Json\ResourceCollection;

class UserCollection extends ResourceCollection
{
    /**
     * 将资源集合转换为数组。
     *
     * @param  \Illuminate\Http\Request  $request
     * @return array
     */
    public function toArray($request)
    {
        return parent::toArray($request);
    }

    /**
     * 获取应该与资源数组一起返回的附加数据。
     *
     * @param  \Illuminate\Http\Request  $request
     * @return array
     */
    public function with($request)
    {
        return [
            'meta' => [
                'key' => 'value',
            ],
        ];
    }
}
```

***

**构造资源时添加元数据**

你还可以在你的路由或控制器中构造资源实例时添加顶级数据。在所有资源上可用的 `additional` 方法接受应添加到资源响应的数据数组：

```php
return (new UserCollection(User::all()->load('roles')))
                ->additional(['meta' => [
                    'key' => 'value',
                ]]);
```

***

## 资源响应

正如你已经读过的那样，资源可以直接从路由和控制器返回：

```php
use App\User;
use App\Http\Resources\User as UserResource;

Route::get('/user', function () {
    return new UserResource(User::find(1));
});
```

***

然而，有时你可能需要在发送到客户机之前定制传出 HTTP 响应。有两种方法可以做到这一点。首先，你可以将 `response` 方法链接到资源上。此方法将返回一个 `Illuminate\Http\Response` 实例，允许你完全控制响应的头部：

```php
use App\User;
use App\Http\Resources\User as UserResource;

Route::get('/user', function () {
    return (new UserResource(User::find(1)))
                ->response()
                ->header('X-Value', 'True');
});
```

***

或者，你可以在资源本身中定义一个 `withResponse` 方法。当资源作为响应中最外层资源返回时，将调用此方法：

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Resources\Json\JsonResource;

class User extends JsonResource
{
    /**
     * 将资源转换为数组。
     *
     * @param  \Illuminate\Http\Request  $request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'id' => $this->id,
        ];
    }

    /**
     * 为资源自定义传出响应。
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Illuminate\Http\Response  $response
     * @return void
     */
    public function withResponse($request, $response)
    {
        $response->header('X-Value', 'True');
    }
}
```

***
