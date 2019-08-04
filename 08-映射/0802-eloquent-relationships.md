# Eloquent：关系

## 简介

数据库表通常是相互关联的。例如，一篇博客文章可能有很多评论，或者一个订单可能与发布它的用户相关。Eloquent 使管理和处理这些关系变得容易，并支持几种不同类型的关系：

* [一对一](https://laravel.com/docs/5.8/eloquent-relationships#one-to-one)
* [一对多](https://laravel.com/docs/5.8/eloquent-relationships#one-to-many)
* [多对多](https://laravel.com/docs/5.8/eloquent-relationships#many-to-many)
* [有一个通过](https://laravel.com/docs/5.8/eloquent-relationships#has-one-through)
* [有多个通过](https://laravel.com/docs/5.8/eloquent-relationships#has-many-through)
* [一对一（多态）](https://laravel.com/docs/5.8/eloquent-relationships#one-to-one-polymorphic-relations)
* [一对多（多态）](https://laravel.com/docs/5.8/eloquent-relationships#one-to-many-polymorphic-relations)
* [多对多（多态）](https://laravel.com/docs/5.8/eloquent-relationships#many-to-many-polymorphic-relations)

## 定义关系

Eloquent 关系被定义为你的 Eloquent 模型类的方法。因为，与 Eloquent 模型本身一样，关系也可以作为强大的 [查询构建器](https://laravel.com/docs/5.8/queries)，将关系定义为方法提供了强大的方法链接和查询能力。例如，我们可以在这个 `posts` 关系上链接其他约束：

```php
$user->posts()->where('active', 1)->get();
```

但是，在深入探究使用关系之前，让我们先学习如何定义每种类型。

### 一对一

一对一的关系是一种非常基本的关系。例如，`User` 模型可能与一个 `Phone` 相关联。要定义此关系，我们在 `User` 模型上放置一个 `phone` 方法。`phone` 方法应调用 `hasOne` 方法并返回其结果：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * 获取与用户关联的电话记录。
     */
    public function phone()
    {
        return $this->hasOne('App\Phone');
    }
}
```

传递给 `hasOne` 方法的第一个参数是相关模型的名称。一旦定义了关系，我们就可以使用 Eloquent 的动态属性检索相关记录。动态属性允许你访问关系方法，就好像它们是在模型上定义的属性一样：

```php
$phone = User::find(1)->phone;
```

Eloquent 根据模型名称来确定关系的外键。在这种情况下，自动假定 `Phone` 模型有一个 `user_id` 外键。如果要覆盖此约定，你可以将第二个参数传递给 `hasOne` 方法：

```php
return $this->hasOne('App\Phone', 'foreign_key');
```

此外，Eloquent 假定外键应有一个与父级的 `id`（或自定义 `$primaryKey`）列匹配的值。换句话说，Eloquent 将在 `Phone` 记录的 `user_id` 列中查找用户 `id` 列的值。如果你希望关系使用 `id` 以外的值，你可以将第三个参数传递到 `hasOne` 方法以指定你的自定义键：

```php
return $this->hasOne('App\Phone', 'foreign_key', 'local_key');
```

#### 定义关系的逆向

因此，我们可以从 `User` 访问 `Phone` 模型。现在，让我们在 `Phone` 模型上定义一个关系，让我们访问拥有手机的 `User`。我们可以使用 `belongsTo` 方法定义 `hasOne` 关系的逆向：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Phone extends Model
{
    /**
     * 获取拥有手机的用户。
     */
    public function user()
    {
        return $this->belongsTo('App\User');
    }
}
```

在上面的示例中，Eloquent 将尝试从 `Phone` 模型中的 `user_id` 与 `User` 模型上的 `id` 匹配。Eloquent 通过检查关系方法的名称并使用 `_id` 为方法名称添加后缀来确定默认外键名称。但是，如果 `Phone` 模型上的外键不是 `user_id`，你可以将自定义一个键名作为第二个参数传递给 `belongsTo` 方法：

```php
/**
 * 获取拥有手机的用户。
 */
public function user()
{
    return $this->belongsTo('App\User', 'foreign_key');
}
```

如果你的父模型不使用 `id` 作为其主键，或者你希望将子模型连接到一个不同的列，你可以将第三个参数传递给 `belongsTo` 方法，以指定父表的自定义键：

```php
/**
 * 获取拥有手机的用户。
 */
public function user()
{
    return $this->belongsTo('App\User', 'foreign_key', 'other_key');
}
```

### 一对多

一对多关系用于定义单个模型拥有任意数量其他模型的关系。例如，一篇博客文章可能有无数的评论。像所有其他 Eloquent 关系一样，一对多关系是通过在你的 Eloquent 模型上放置一个方法来定义的：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Post extends Model
{
    /**
     * 获取博客文章的评论。
     */
    public function comments()
    {
        return $this->hasMany('App\Comment');
    }
}
```

### 多对多

### 有一个通过

### 有多个通过

## 多态关系

## 查询关系

## 预先加载

## 插入 & 更新相关的模型

## 触摸父时间戳
