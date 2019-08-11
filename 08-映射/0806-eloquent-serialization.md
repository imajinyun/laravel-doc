# Eloquent：序列化

***

## 简介

构建 JSON API 时，你通常将需要将你的模型和关系转换为数组或 JSON。Eloquent 包括用于进行这些转换的便捷方法，以及控制在你的序列化中包含哪些属性。

## 序列化模型 & 集合

### 序列化为数组

要将模型及其加载的 [关系](https://laravel.com/docs/5.8/eloquent-relationships) 转换为数组，你应使用 `toArray` 方法。此方法是递归的，因此所有属性和所有关系（包括关系的关系）都将转换为数组：

```php
$user = App\User::with('roles')->first();

return $user->toArray();
```

***

要仅将模型的属性转换为数组，使用 `attributesToArray` 方法：

```php
$user = App\User::first();

return $user->attributesToArray();
```

***

你还可以转换整个模型的 [集合](https://laravel.com/docs/5.8/eloquent-collections) 为数组：

```php
$users = App\User::all();

return $users->toArray();
```

### 序列化到 JSON

要将模型转换为 JSON，你应该使用 `toJson` 方法。与 `toArray` 一样，`toJson` 方法是递归的，因此所有属性和关系都将转换为 JSON。你还可以指定 [PHP 支持的](https://secure.php.net/manual/en/function.json-encode.php) JSON 编码选项：

```php
$user = App\User::find(1);

return $user->toJson();

return $user->toJson(JSON_PRETTY_PRINT);
```

***

或者，你可以将模型或集合转换为字符串，字符串将自动调用模型或集合上的 `toJson` 方法：

```php
$user = App\User::find(1);

return (string) $user;
```

***

由于模型和集合在转换为字符串时转换为 JSON，所以可以直接从你的应用程序的路由或控制器返回 Eloquent 对象：

```php
Route::get('users', function () {
    return App\User::all();
});
```

***

**关系**

当 Eloquent 模型转换为 JSON 时，其加载的关系将自动作为属性包含在 JSON 对象中。此外，尽管 Eloquent 关系方法是使用『驼峰式』定义的，但是关系的 JSON 属性将是『蛇形式』命名。

## 从 JSON 中隐藏属性

有时你可能希望限制你的模型数组或 JSON 表示中包含的属性（如：密码）。为此，向你的模型添加 `$hidden` 属性：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * 应该为数组隐藏的属性。
     *
     * @var array
     */
    protected $hidden = ['password'];
}
```

***

{% hint style="danger" %}

隐藏关系时，使用关系的方法名称。

{% endhint %}

或者，你可以使用 `visible` 属性来定义应包含在你的模型的数组和 JSON 表示中的白名单属性。将模型转换为数组或 JSON 时，将隐藏所有其他属性：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * 应该在数组中显示的属性。
     *
     * @var array
     */
    protected $visible = ['first_name', 'last_name'];
}
```

***

**临时修改属性显示**

如果你希望在给定的模型实例上显示一些典型的隐藏属性，可以使用 `makeVisible` 方法。`makeVisible` 方法返回方便方法链接的模型实例：

```php
return $user->makeVisible('attribute')->toArray();
```

***

同样，如果你希望在给定的模型实例上隐藏一些典型的显示属性，你可以使用 `makeHidden` 方法。

```php
return $user->makeHidden('attribute')->toArray();
```

## 追加值到 JSON

有时，在将模型转换为数组或 JSON 时，你可能希望添加的属性在你的数据库中没有对应的列。为此，首先为值定义一个 [访问器](https://laravel.com/docs/5.8/eloquent-mutators)：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * Get the administrator flag for the user.
     * 获取用户的管理员标志。
     *
     * @return bool
     */
    public function getIsAdminAttribute()
    {
        return $this->attributes['admin'] == 'yes';
    }
}
```

***

创建访问器后，将属性名称添加到模型的 `appends` 属性中。注意，属性名称通常在『蛇形命名』中引用，即使访问器是使用『驼峰命名』定义的：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * 要追加到模型的数组形式的访问器。
     *
     * @var array
     */
    protected $appends = ['is_admin'];
}
```

## 日期序列化

**自定义每个属性的日期格式**

**通过 Carbon 全局定制**
