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

记住，Eloquent 将自动确定 `Comment` 模型上的正确外键列。按照惯例，Eloquent 将采用拥有模型的『蛇形』名称，并以 `_id` 为后缀。因此，对于此示例，Eloquent 将假定 `Comment` 模型上的外键是 `post_id`。

一旦定义了关系，我们就可以通过访问 `comments` 属性来访问评论集合。记住，由于 Eloquent 提供了『动态属性』，我们可以像访问模型中的属性一样访问关系方法：

```php
$comments = App\Post::find(1)->comments;

foreach ($comments as $comment) {
    //
}
```

由于所有关系也充当查询构建器，因此你可以通过调用 `comments` 方法并继续将条件链接到查询来进一步添加对检索的评论的更多约束：

```php
$comment = App\Post::find(1)->comments()->where('title', 'foo')->first();
```

与 `hasOne` 方法一样，你也可以通过将其他参数传递给 `hasMany` 方法来覆盖外键和本地键：

```php
return $this->hasMany('App\Comment', 'foreign_key');

return $this->hasMany('App\Comment', 'foreign_key', 'local_key');
```

### 一对多（逆向）

现在我们可以访问所有帖子的评论，让我们定义一个关系，允许评论访问其父帖子。要定义 `hasMany` 关系的逆关系，在子模型上定义一个调用 `belongsTo` 方法的关系函数：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Comment extends Model
{
    /**
     * 获取拥有评论的帖子。
     */
    public function post()
    {
        return $this->belongsTo('App\Post');
    }
}
```

一旦定义了关系，我们就可以通过访问 `post`『动态属性』来检索 `Comment` 的 `Post` 模型：

```php
$comment = App\Comment::find(1);

echo $comment->post->title;
```

在上面的示例中，Eloquent 将尝试将 `Comment` 模型中的 `post_id` 与 `Post` 模型上的 `id` 匹配。Eloquent 通过检查关系方法的名称并带一个 `_` 的方法后缀再加上主键列的名称来确定默认外键名称。但是，如果 `Comment` 模型上的外键不是 `post_id`，你可以将自定义键名作为第二个参数传递给 `belongsTo` 方法：

```php
/**
 * 获取拥有评论的帖子。
 */
public function post()
{
    return $this->belongsTo('App\Post', 'foreign_key');
}
```

如果你的父模型不使用 `id` 作为其主键，或者你希望将子模型连接到不同的列，你可以将第三个参数传递给 `belongsTo` 方法，以指定你的父表的自定义键：

```php
/**
 * 获取拥有评论的帖子。
 */
public function post()
{
    return $this->belongsTo('App\Post', 'foreign_key', 'other_key');
}
```

### 多对多

多对多关系比 `hasOne` 和 `hasMany` 关系略微复杂。这种关系的一个示例是具有许多角色的用户，其中角色也由其他用户共享。例如，许多用户可能具有『管理员』的角色。要定义此关系，需要三个数据库表：`users`、`roles` 和 `role_user`。`role_user` 表派生自相关模型名称的字母顺序，并包含 `user_id` 和 `role_id` 列。

通过编写返回 `belongsToMany` 方法结果的方法来定义多对多关系。例如，让我们在 `User` 模型上定义 `roles` 方法：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * 属于用户的角色。
     */
    public function roles()
    {
        return $this->belongsToMany('App\Role');
    }
}
```

一旦定义了关系，你就可以使用 `roles` 动态属性访问你的用户的角色：

```php
$user = App\User::find(1);

foreach ($user->roles as $role) {
    //
}
```

与所有其他关系类型一样，你可以调用 `roles` 方法继续将查询约束链接到关系上：

```php
$roles = App\User::find(1)->roles()->orderBy('name')->get();
```

如前所述，为确定关系连接表的表名，Eloquent 将按字母顺序连接两个相关的模型名称。但是，你可以自由地覆盖此约定。你可以通过将第二个参数传递给 `belongsToMany` 方法来执行此操作：

```php
return $this->belongsToMany('App\Role', 'role_user');
```

除了自定义连接表的名称之外，你还可以通过将其他参数传递给 `belongsToMany` 方法来自定义表上键的列名。第三个参数是你定义关系的模型的外键名称，而第四个参数是你要连接的模型的外键名称：

```php
return $this->belongsToMany('App\Role', 'role_user', 'user_id', 'role_id');
```

#### 定义关系的逆向

要定义多对多关系的逆关系，可以在你的相关模型上再次调用 `belongsToMany`。要继续我们的用户角色示例，让我们在 `Role` 模型上定义 `users` 方法：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Role extends Model
{
    /**
     * 属于该角色的用户。
     */
    public function users()
    {
        return $this->belongsToMany('App\User');
    }
}
```

如您所见，除了引用 `App\User` 模型之外，该关系的定义与其 `User` 对象完全相同。由于我们正在重用 `belongsToMany` 方法，因此在定义多对多关系的逆关系时，所有常用的表和键自定义选项都可用。

#### 检索中间表列

正如你已经了解的那样，处理多对多关系需要存在中间表。Eloquent 提供了一些与此表交互的非常有用的方法。例如，假设我们的 `User` 对象有许多与其相关的 `Role` 对象。访问此关系后，我们可以使用模型上的 `pivot` 属性访问中间表：

```php
$user = App\User::find(1);

foreach ($user->roles as $role) {
    echo $role->pivot->created_at;
}
```

注意，我们检索的每个 `Role` 模型都自动分配了一个 `pivot` 属性。该属性包含一个表示中间表的模型，可以像其他任何 Eloquent 模型一样使用。

默认情况下，只有模型键才会出现在 `pivot` 对象上。如果你的中转表包含额外的属性，你必须在定义关系时指定它们：

```php
return $this->belongsToMany('App\Role')->withPivot('column1', 'column2');
```

如果希望中转表自动维护 `created_at` 和 `updated_at` 时间戳，在关系定义上使用 `withTimestamp` 方法：

```php
return $this->belongsToMany('App\Role')->withTimestamps();
```

#### 自定义 `pivot` 属性名称

如前所述，可以使用 `pivot` 属性在模型上访问中间表中的属性。但是，你可以自由定制此属性的名称，以更好地反映其在应用程序中的意图。

例如，如果你的应用程序包含可能订阅播客的用户，那么用户和播客之间可能存在多对多关系。如果是这种情况，你可能希望将中间表访问器重命名为 `subscription` 而不是 `pivot`。这可以在定义关系时使用 `as` 方法来完成：

```php
return $this->belongsToMany('App\Podcast')
                ->as('subscription')
                ->withTimestamps();
```

一旦完成此操作，你可以使用自定义名称访问中间表数据：

```php
$users = User::with('podcasts')->get();

foreach ($users->flatMap->podcasts as $podcast) {
    echo $podcast->subscription->created_at;
}
```

#### 通过中间表列过滤关系

在定义关系时，你还可以使用 `wherePivot` 和 `wherePivotIn` 方法来过滤通过 `belongsToMany` 返回的结果：

```php
return $this->belongsToMany('App\Role')->wherePivot('approved', 1);

return $this->belongsToMany('App\Role')->wherePivotIn('priority', [1, 2]);
```

### 定义自定义中间表模型

如果你想自定义模型来表示你的关系的中间表，你可以在定义关系时调用 `using` 方法。自定义多对多中转模型应继承 `Illuminate\Database\Eloquent\Relations\Pivot` 类，而自定义多态多对多中转模型应继承 `Illuminate\Database\Eloquent\Relations\MorphPivot` 类。例如，我们可以定义一个使用自定义 `RoleUser` 中转模型的 `Role`：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Role extends Model
{
    /**
     * 属于该角色的用户。
     */
    public function users()
    {
        return $this->belongsToMany('App\User')->using('App\RoleUser');
    }
}
```

在定义 `RoleUser` 模型时，我们将继承 `Pivot` 类：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Relations\Pivot;

class RoleUser extends Pivot
{
    //
}
```

你可以将 `using` 和 `withPivot` 结合使用，以便从中间表中检索列。例如，你可以通过将列名称传递给 `withPivot` 方法从 `RoleUser` 中转表中检索 `created_by` 和 `updated_by` 列：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Role extends Model
{
    /**
     * 属于该角色的用户。
     */
    public function users()
    {
        return $this->belongsToMany('App\User')
                        ->using('App\RoleUser')
                        ->withPivot([
                            'created_by',
                            'updated_by'
                        ]);
    }
}
```

{% hint style="danger" %}

**注意**：中转模型可能不使用 `SoftDeletes` 特性。如果你需要软删除中转记录，考虑将你的中转模型转换为实际的 Eloquent 模型。

{% endhint %}

#### 自定义中转模型和递增 ID

如果你已经定义使用自定义中转模型的多对多关系，并且该中转模型具有自动递增的主键，你应确保自定义中转模型类定义一个设置为 `true` 的 `incrementing` 属性。

```php
/**
 * 指示 ID 是否自动递增。
 *
 * @var bool
 */
public $incrementing = true;
```

### 有一个通过

『单向』关系通过单个中间关系链接模型。例如，如果每个供应商有一个用户，并且每个用户与一个用户历史记录相关联，那么供应商模型可以通过用户访问用户的历史。让我们看看定义这种关系所需的数据库表：

```php
users
    id - integer
    supplier_id - integer

suppliers
    id - integer

history
    id - integer
    user_id - integer
```

虽然 `history` 表不包含 `supplier_id` 列，但 `hasOneThrough` 关系可以提供对供应商模型的用户历史记录的访问权限。现在我们已经检查了关系的表结构，让我们在 `Supplier` 模型上定义它：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Supplier extends Model
{
    /**
     * 获取用户的历史记录。
     */
    public function userHistory()
    {
        return $this->hasOneThrough('App\History', 'App\User');
    }
}
```

传递给 `hasOneThrough` 方法的第一个参数是我们希望访问的最终模型的名称，而第二个参数是中间模型的名称。

执行关系查询时将使用典型的 Eloquent 外键约定。如果要自定义关系的键，可以将它们作为 `hasOneThrough` 方法的第三个和第四个参数传递。第三个参数是中间模型上的外键名称。第四个参数是最终模型上的外键名称。第五个参数是本地键，而第六个参数是中间模型的本地键：

```php
class Supplier extends Model
{
    /**
     * 获取用户的历史记录。
     */
    public function userHistory()
    {
        return $this->hasOneThrough(
            'App\History',
            'App\User',
            'supplier_id', // 用户表的外键...
            'user_id', // 历史表的外键...
            'id', // 供应商表的本地键...
            'id' // 用户表的本地键...
        );
    }
}
```

### 有多个通过

『有多个通过』关系为通过中间关系访问远程关系提供了方便的快捷方式。例如，一个 `Country` 模型可能通过一个中间 `User` 模型有许多 `Post` 模型。在本例中，你可以轻松地收集给定国家的所有博客文章。让我们看看定义这种关系所需的表：

```bash
countries
    id - integer
    name - string

users
    id - integer
    country_id - integer
    name - string

posts
    id - integer
    user_id - integer
    title - string
```

虽然 `posts` 不包含 `country_id` 列，但 `hasManyThrough` 关系通过 `$country->posts` 提供对一个国家的帖子的访问。要执行此查询，Eloquent 会检查中间用户表上的 `country_id`。找到匹配的用户 ID 后，它们用于查询 `posts` 表。

现在我们已经检查了关系的表结构，让我们在 `Country` 模型上定义它：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Country extends Model
{
    /**
     * 获取该国家的所有帖子。
     */
    public function posts()
    {
        return $this->hasManyThrough('App\Post', 'App\User');
    }
}
```

传递给 `hasManyThrough` 方法的第一个参数是我们希望访问的最终模型的名称，而第二个参数是中间模型的名称。

执行关系查询时将使用典型的 Eloquent 外键约定。如果要自定义关系的键，可以将它们作为 `hasManyThrough` 方法的第三个和第四个参数传递。第三个参数是中间模型上的外键名称。第四个参数是最终模型上的外键名称。第五个参数是本地键，而第六个参数是中间模型的本地键：

```php
class Country extends Model
{
    public function posts()
    {
        return $this->hasManyThrough(
            'App\Post',
            'App\User',
            'country_id', // 用户表上的外键...
            'user_id', // 帖子表上的外键...
            'id', // 国家表上的本地键...
            'id' // 用户表上的本地键...
        );
    }
}
```

## 多态关系

多态关系允许目标模型使用单个关联属于多种类型的模型。

### 一对一（多态）

#### 表结构

一对一的多态关系类似于简单的一对一关系；但是，目标模型可以属于单个关联上的多种类型的模型。例如，博客 `Post` 和 `User` 可以共享与 `Image` 模型的多态关系。使用一对一的多态关系允许让你拥有一个用于博客帖子和用户帐户的唯一图像列表。首先，让我们检查一下表结构：

```php
posts
    id - integer
    name - string

users
    id - integer
    name - string

images
    id - integer
    url - string
    imageable_id - integer
    imageable_type - string
```

注意 `images` 表上的 `imageable_id` 和 `imageable_type` 列。`imageable_id` 列将包含帖子或用户的 ID 值，而 `imageable_type` 列将包含父模型的类名。Eloquent 使用 `imageable_type` 列来确定在访问 `imageable` 关系时要返回父模型的哪个『类型』。

#### 模型结构

接下来，让我们检查构建此关系所需的模型定义：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Image extends Model
{
    /**
     * 获取拥有的可成像图片的模型。
     */
    public function imageable()
    {
        return $this->morphTo();
    }
}

class Post extends Model
{
    /**
     * 获取帖子的图片。
     */
    public function image()
    {
        return $this->morphOne('App\Image', 'imageable');
    }
}

class User extends Model
{
    /**
     * 获取用户的图片。
     */
    public function image()
    {
        return $this->morphOne('App\Image', 'imageable');
    }
}
```

#### 检索关系

一旦定义了你的数据库表和模型，你就可以通过你的模型访问关系。例如，要检索帖子的图像，我们可以使用 `image` 动态属性：

```php
$post = App\Post::find(1);

$image = $post->image;
```

你还可以通过访问执行对 `morphTo` 调用的方法的名称，从多态模型中检索父类。在我们的例子中，这是 `Image` 模型上的 `imageable` 方法。因此，我们将作为动态属性访问那个方法：

```php
$image = App\Image::find(1);

$imageable = $image->imageable;
```

`Image` 模型上的 `imageable` 关系将返回 `Post` 或 `User` 实例，具体取决于拥有图像的模型类型。

### 一对多（多态）

一对多多态关系类似于简单的一对多关系；然而，目标模型可以属于单个关联上的多个模型类型。例如，假设应用程序的用户可以对帖子和视频进行『评论』。使用多态关系，你可以为这两种场景使用一个 `comments` 表。首先，让我们检查构建此关系所需的表结构：

#### 表结构

```bash
posts
    id - integer
    title - string
    body - text

videos
    id - integer
    title - string
    url - string

comments
    id - integer
    body - text
    commentable_id - integer
    commentable_type - string
```

#### 模型结构

接下来，让我们检查构建此关系所需的模型定义：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Comment extends Model
{
    /**
     * 获得拥有可评论的模型。
     */
    public function commentable()
    {
        return $this->morphTo();
    }
}

class Post extends Model
{
    /**
     * 获取所有帖子的评论。
     */
    public function comments()
    {
        return $this->morphMany('App\Comment', 'commentable');
    }
}

class Video extends Model
{
    /**
     * 获取所有视频的评论。
     */
    public function comments()
    {
        return $this->morphMany('App\Comment', 'commentable');
    }
}
```

#### 检索关系

一旦定义了数据库表和模型，你就可以通过你的模型访问关系。例如，要访问一篇文章的所有评论，我们可以使用 `comments` 动态属性：

```php
$post = App\Post::find(1);

foreach ($post->comments as $comment) {
    //
}
```

你还可以通过访问执行对 `morphTo` 调用的方法的名称，从多态模型中检索多态关系的所有者。在我们的例子中，这是 `Comment` 模型上的 `commentable` 方法。因此，我们将作为动态属性访问那个方法：

```php
$comment = App\Comment::find(1);

$commentable = $comment->commentable;
```

`Comment` 模型上的 `commentable` 关系将返回 `Post` 或 `Video` 实例，具体取决于哪种类型的模型拥有该注释。

### 多对多（多态）

多对多的多态关系比 `morphOne` 和 `morphMany` 关系稍微复杂一些。例如，博客 `Post` 和 `Video` 模型可以与 `Tag` 模型共享多态关系。使用多对多多态关系，允许你有一个在博客文章和视频中共享的唯一标记列表。首先，让我们检查一下表结构：

#### 表结构

```php
posts
    id - integer
    name - string

videos
    id - integer
    name - string

tags
    id - integer
    name - string

taggables
    tag_id - integer
    taggable_id - integer
    taggable_type - string
```

#### 模型结构

接下来，我们准备定义模型上的关系。`Post` 和 `Video` 模型都有一个 `tags` 方法，可以调用基本 Eloquent 类的 `morphToMany` 方法：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Post extends Model
{
    /**
     * 获取帖子的所有标签。
     */
    public function tags()
    {
        return $this->morphToMany('App\Tag', 'taggable');
    }
}
```

#### 定义关系的逆关系

接下来，在 `Tag` 模型上，你应该为每个相关模型定义一个方法。 因此，对于此示例，我们将定义 `posts` 方法和 `videos` 方法：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Tag extends Model
{
    /**
     * 获取分配了此标记的所有帖子。
     */
    public function posts()
    {
        return $this->morphedByMany('App\Post', 'taggable');
    }

    /**
     * 获取分配了此标记的所有视频。
     */
    public function videos()
    {
        return $this->morphedByMany('App\Video', 'taggable');
    }
}
```

#### 检索关系

一旦定义了数据库表和模型，你就可以通过你的模型访问关系。例如，要访问一个帖子的所有标记，可以使用 `tags` 动态属性：

```php
$post = App\Post::find(1);

foreach ($post->tags as $tag) {
    //
}
```

***

你还可以通过访问执行对 `morphedByMany` 调用的方法的名称，从多态模型中检索多态关系的所有者。在我们的例子中，这是 `Tag` 模型上的 `posts` 或 `videos` 方法。因此，你将作为动态属性访问这些方法：

```php
$tag = App\Tag::find(1);

foreach ($tag->videos as $video) {
    //
}
```

### 自定义多态类型

默认情况下，Laravel 将使用完全限定的类名来存储相关模型的类型。例如，给定上面的一对多示例，其中 `Comment` 可以属于 `Post` 或 `Video`，默认的 `commentable_type` 将分别是 `App\Post` 或 `App\Video`。但是，你可能希望将你的数据库从你的应用程序的内部结构中解耦。在这种情况下，你可以定义『变形映射』来指示 Eloquent 为每个模型使用自定义名称而不是类名：

```php
use Illuminate\Database\Eloquent\Relations\Relation;

Relation::morphMap([
    'posts' => 'App\Post',
    'videos' => 'App\Video',
]);
```

你可以在 `AppServiceProvider` 的 `boot` 方法中注册 `morphMap`，或者根据你的需要创建单独的服务提供者。

{% hint style="danger" %}

向现有应用程序添加『变形映射』时，在你的数据库中仍包含完全限定类的每个可变形 `* _type` 列值都需要转换为其『映射』名称。

{% endhint %}

## 查询关系

由于所有类型的 Eloquent 关系都是通过方法定义的，因此你可以调用这些方法来获取关系的实例，而无需实际执行关系查询。此外，所有类型的 Eloquent 关系也可用作 [查询构建器](https://laravel.com/docs/5.8/queries)，允许你在最终针对你的数据库执行 SQL 之前继续将约束链接到关系查询上。

例如，想象一个博客系统，其中一个 `User` 模型有许多相关的 `Post` 模型：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * 获取用户的所有帖子。
     */
    public function posts()
    {
        return $this->hasMany('App\Post');
    }
}
```

你可以查询 `posts` 关系并为关系像这样添加其他约束：

```php
$user = App\User::find(1);

$user->posts()->where('active', 1)->get();
```

***

你可以在关系上使用任何 [查询构建器](https://laravel.com/docs/5.8/queries) 方法，因此请务必浏览查询构建器文档以了解可用的所有方法。

**关系之后的链接 `orWhere` 子句**

正如上面的示例所示，在查询关系时，可以自由地向关系添加额外的约束。但是，在将 `orWhere` 子句链接到关系上时要小心，因为 `orWhere` 子句将关系约束在逻辑分组的同一级别上：

```php
$user->posts()
        ->where('active', 1)
        ->orWhere('votes', '>=', 100)
        ->get();

// select * from posts
// where user_id = ? and active = 1 or votes >= 100
```

在大多数情况下，你可能打算使用 [约束组](https://laravel.com/docs/5.8/queries#parameter-grouping) 对括号中的条件检查进行逻辑分组：

```php
use Illuminate\Database\Eloquent\Builder;

$user->posts()
        ->where(function (Builder $query) {
            return $query->where('active', 1)
                         ->orWhere('votes', '>=', 100);
        })
        ->get();

// select * from posts
// where user_id = ? and (active = 1 or votes >= 100)
```

***

### 关系方法 Vs 动态属性

如果您不需要为 Eloquent 关系查询添加其他约束，你可以像访问属性一样访问该关系。例如，继续使用我们的 `User` 和 `Post` 示例模型，我们可以像这样访问一个用户的所有帖子：

```php
$user = App\User::find(1);

foreach ($user->posts as $post) {
    //
}
```

动态属性是『延迟加载』，这意味着它们只会在你实际访问它们时加载它们的关系数据。因此，开发人员经常使用 [预先加载](https://laravel.com/docs/5.8/eloquent-relationships#eager-loading) 来预加载他们知道在加载模型后将被访问的关系。预先加载可显著地减少必须执行以加载模型关系的 SQL 查询。

### 查询关系存在

在访问模型的记录时，你可能希望基于关系的存在来限制你的结果。例如，假设你想检索所有至少有一条评论的博客文章。为此，你可以将关系的名称传递给 `has` 和 `orHas` 方法：

```php
// 检索至少有一条评论的所有帖子...
$posts = App\Post::has('comments')->get();
```

你还可以指定运算符和计数以进一步自定义查询：

```php
// 检索包含三个或更多评论的所有帖子...
$posts = App\Post::has('comments', '>=', 3)->get();
```

嵌套的 `has` 语句也可以使用『点』表示法构造。例如，你可以检索至少有一条评论和投票的所有帖子：

```php
// 检索至少有一条带有投票评论的所有帖子...
$posts = App\Post::has('comments.votes')->get();
```

如果你需要更强大的功能，你可以使用 `whereHas` 和 `orWhereHas` 方法去放你的『where』条件到你的 `has` 查询中。这些方法允许你向关系约束添加自定义约束，例如检查评论的内容：

```php
use Illuminate\Database\Eloquent\Builder;

// 检索包含至少一条评论的帖子，其中包含像 foo％ 这样的字词...
$posts = App\Post::whereHas('comments', function (Builder $query) {
    $query->where('content', 'like', 'foo%');
})->get();

// 检索包含至少十条评论的帖子，其中包含像 foo％ 这样的字词...
$posts = App\Post::whereHas('comments', function (Builder $query) {
    $query->where('content', 'like', 'foo%');
}, '>=', 10)->get();
```

### 查询关系缺失

访问模型的记录时，你可能希望根据缺失的关系来限制你的结果。例如，假设你要检索所有 **没有** 任何评论的博客帖子。为此，你可以将关系的名称传递给 `doesntHave` 和 `orDoesntHave` 方法：

```php
$posts = App\Post::doesntHave('comments')->get();
```

如果你需要更强大的功能，你可以使用 `whereDoesntHave` 和 `orWhereDoesntHave` 方法将『where』条件放在你的 `doesntHave` 查询上。这些方法允许你向关系约束添加自定义约束，例如检查评论的内容：

```php
use Illuminate\Database\Eloquent\Builder;

$posts = App\Post::whereDoesntHave('comments', function (Builder $query) {
    $query->where('content', 'like', 'foo%');
})->get();
```

你可以使用『点』表示法来执行针对嵌套关系的查询。例如，以下查询将检索所有包含未被禁止的作者的评论的帖子：

```php
use Illuminate\Database\Eloquent\Builder;

$posts = App\Post::whereDoesntHave('comments.author', function (Builder $query) {
    $query->where('banned', 1);
})->get();
```

### 查询多态关系

要查询 `MorphTo` 关系的存在，你可以使用 `whereHasMorph` 方法及其相应的方法：

```php
use Illuminate\Database\Eloquent\Builder;

// 检索标题像 foo％ 的与帖子或视频相关的评论...
$comments = App\Comment::whereHasMorph(
    'commentable',
    ['App\Post', 'App\Video'],
    function (Builder $query) {
        $query->where('title', 'like', 'foo%');
    }
)->get();

// 检索标题不像 foo% 的与帖子关联的评论...
$comments = App\Comment::whereDoesntHaveMorph(
    'commentable',
    'App\Post',
    function (Builder $query) {
        $query->where('title', 'like', 'foo%');
    }
)->get();
```

你可以使用 `$type` 参数根据相关模型添加不同的约束：

```php
use Illuminate\Database\Eloquent\Builder;

$comments = App\Comment::whereHasMorph(
    'commentable',
    ['App\Post', 'App\Video'],
    function (Builder $query, $type) {
        $query->where('title', 'like', 'foo%');

        if ($type === 'App\Post') {
            $query->orWhere('content', 'like', 'foo%');
        }
    }
)->get();
```

你可以提供 `*` 作为通配符，让 Laravel 从数据库中检索所有可能的多态类型，而不是传递可能的多态模型数组。Laravel 将执行额外的查询为执行此操作：

```php
use Illuminate\Database\Eloquent\Builder;

$comments = App\Comment::whereHasMorph('commentable', '*', function (Builder $query) {
    $query->where('title', 'like', 'foo%');
})->get();
```

### 统计相关模型

如果你想计算关系中的结果数而不实际加载它们，你可以使用 `withCount` 方法，该方法会在你的结果模型上放置 `{relation} _count` 列。例如：

```php
$posts = App\Post::withCount('comments')->get();

foreach ($posts as $post) {
    echo $post->comments_count;
}
```

你可以为多个关系添加『计数』以及为查询添加约束：

```php
use Illuminate\Database\Eloquent\Builder;

$posts = App\Post::withCount(['votes', 'comments' => function (Builder $query) {
    $query->where('content', 'like', 'foo%');
}])->get();

echo $posts[0]->votes_count;
echo $posts[0]->comments_count;
```

你也可以为关系计数结果添加别名，允许对同一关系进行多次计数：

```php
use Illuminate\Database\Eloquent\Builder;

$posts = App\Post::withCount([
    'comments',
    'comments as pending_comments_count' => function (Builder $query) {
        $query->where('approved', false);
    }
])->get();

echo $posts[0]->comments_count;

echo $posts[0]->pending_comments_count;
```

如果你将 `withCount` 与 `select` 语句结合使用，确保你在 `select` 方法之后调用 `withCount`：

```php
$posts = App\Post::select(['title', 'body'])->withCount('comments')->get();

echo $posts[0]->title;
echo $posts[0]->body;
echo $posts[0]->comments_count;
```

## 预先加载

当访问 Eloquent 关系作为属性时，关系数据是『延迟加载』。这意味着在你首次访问该属性之前，实际上不会加载关系数据。但是，Eloquent 可以在你查询父模型时『预告加载』关系。预先加载缓解了 N + 1 查询问题。要说明 N + 1 查询问题，考虑与 `Author` 相关的 `Book` 模型：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Book extends Model
{
    /**
     * 获取写这本书的作者。
     */
    public function author()
    {
        return $this->belongsTo('App\Author');
    }
}
```

现在，让我们检索所有书籍及其作者：

```php
$books = App\Book::all();

foreach ($books as $book) {
    echo $book->author->name;
}
```

这个循环将执行一个查询来检索表上的所有书籍，然后对每本书执行另一个查询来检索作者。因此，如果我们有 25 本书，这个循环将运行 26 个查询：1 个查询原始图书，另外 25 个查询检索每本书的作者。

值得庆幸的是，我们可以使用预先加载将此操作减少到只有 2 个查询。查询时，你可以使用 `with` 方法指定应该预先加载哪些关系：

```php
$books = App\Book::with('author')->get();

foreach ($books as $book) {
    echo $book->author->name;
}
```

对于此操作，将只执行两个查询：

```sql
select * from books

select * from authors where id in (1, 2, 3, 4, 5, ...)
```

**预先加载多个关系**

有时你可能需要在单个操作中预先加载几个不同的关系。为此，只需将其他参数传递给 `with` 方法：

```php
$books = App\Book::with(['author', 'publisher'])->get();
```

***

**嵌套预先加载**

要预先加载嵌套关系，可以使用『点』语法。例如，让我们在一个 Eloquent 声明中预先载入该书的所有作者和所有作者的个人联系：

```php
$books = App\Book::with('author.contacts')->get();
```

***

**嵌套的预先加载 `morphTo` 关系**

如果希望预先加载 `morphTo` 关系，以及该关系可能返回的各种实体上的嵌套关系，你可以将 `with` 方法与 `morphTo` 关系的 `morphWith` 方法结合使用。为了帮助说明这个方法，让我们考虑下面的模型：

```php
<?php

use Illuminate\Database\Eloquent\Model;

class ActivityFeed extends Model
{
    /**
     * 获取活动提要记录的父记录。
     */
    public function parentable()
    {
        return $this->morphTo();
    }
}
```

在这个例子中，我们假设 `Event`、`Photo` 和 `Post` 模型可以创建 `ActivityFeed` 模型。此外，我们假设 `Event` 模型属于 `Calendar` 模型，`Photo` 模型与 `Tag` 模型相关联，而 `Post` 模型属于 `Author` 模型。

使用这些模型定义和关系，我们可以检索 `ActivityFeed` 模型实例并预先加载所有 `parentable` 模型及其各自的嵌套关系：

```php
use Illuminate\Database\Eloquent\Relations\MorphTo;

$activities = ActivityFeed::query()
    ->with(['parentable' => function (MorphTo $morphTo) {
        $morphTo->morphWith([
            Event::class => ['calendar'],
            Photo::class => ['tags'],
            Post::class => ['author'],
        ]);
    }])->get();
```

***

**预先加载特定的列**

你可能并不总是需要检索关系中的每一列。因此，Eloquent 允许你指定要检索的关系的哪些列：

```php
$books = App\Book::with('author:id,name')->get();
```

{% hint style="danger" %}

使用此功能时，你应始终在你希望要检索的列的列表中包含 `id` 列和任何相关的外键列。

{% endhint %}

***

**默认情况下预先加载**

有时你可能希望在检索模型时始终加载某些关系。要完成此任务，你可以在模型上定义 `$with` 属性：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Book extends Model
{
    /**
     * 应始终加载的关系。
     *
     * @var array
     */
    protected $with = ['author'];

    /**
     * 获取写这本书的作者。
     */
    public function author()
    {
        return $this->belongsTo('App\Author');
    }
}
```

如果要从单个查询的 `$with` 属性中删除条目，可以使用 `without` 方法：

```php
$books = App\Book::without('author')->get();
```

***

### 约束预先加载

有时你可能希望预先加载关系，但也为预先加载查询指定其他查询条件。这儿有一个例子：

```php
$users = App\User::with(['posts' => function ($query) {
    $query->where('title', 'like', '%first%');
}])->get();
```

在这个例子中，Eloquent 只会预先加载帖子的 `title` 列包含 `first` 单词的帖子。你可以调用其他 [查询构建器](https://laravel.com/docs/5.8/queries) 方法来进一步自定义预先加载操作：

```php
$users = App\User::with(['posts' => function ($query) {
    $query->orderBy('created_at', 'desc');
}])->get();
```

{% hint style="danger" %}

在约束预先加载时，可能不使用 `limit` 和 `take` 查询构建器方法。

{% endhint %}

### 懒预先加载

有时你可能需要在已经检索的父模型之后预先加载关系。例如，如果你需要动态决定是否加载相关模型，这可能很有用：

```php
$books = App\Book::all();

if ($someCondition) {
    $books->load('author', 'publisher');
}
```

如果你需要在预先加载的查询上设置其他查询约束，你可以传递一个由你希望加载的关系键的数组。该数组的值应该是接收查询实例的 `Closure` 实例：

```php
$books->load(['author' => function ($query) {
    $query->orderBy('published_date', 'asc');
}]);
```

要仅在尚未加载关系时加载关系，使用 `loadMissing` 方法：

```php
public function format(Book $book)
{
    $book->loadMissing('author');

    return [
        'name' => $book->name,
        'author' => $book->author->name
    ];
}
```

***

**嵌套的懒预先加载 & `morphTo`**

## 插入 & 更新相关的模型

## 触及父时间戳
