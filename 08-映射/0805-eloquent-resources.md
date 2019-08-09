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
