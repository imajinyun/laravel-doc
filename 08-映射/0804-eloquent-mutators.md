# Eloquent：存取器

## 简介

访问器和存取器允许你在模型实例上检索或设置 Eloquent 属性值时对其进行格式化。例如，你可能希望使用 [Laravel 加密器](https://laravel.com/docs/5.8/encryption) 对存储在数据库中时对值进行加密，然后当你在 Eloquent 模型上访问该属性时自动解密该属性。

除了自定义访问器和存取器之外，Eloquent 还可以自动将日期字段转换为 [Carbon](https://github.com/briannesbitt/Carbon) 实例，甚至可以将 [文本字段转换为 JSON](https://laravel.com/docs/5.8/eloquent-mutators#attribute-casting)。

## 访问器 & 存取器

### 定义一个访问器

要定义访问器，在你的模型上创建一个 `getFooAttribute` 方法，其中 `Foo` 是你要访问的列的『驼峰式』名称。在此示例中，我们将为 `first_name` 属性定义一个访问器。尝试检索 `first_name` 属性的值时，Eloquent 会自动调用访问器：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * 获取用户的名字。
     *
     * @param  string  $value
     * @return string
     */
    public function getFirstNameAttribute($value)
    {
        return ucfirst($value);
    }
}
```

如你所见，列的原始值将传递给访问器，允许你操作并返回值。要访问访问器的值，你可以访问模型实例上的 `first_name` 属性：

```php
$user = App\User::find(1);

$firstName = $user->first_name;
```

你还可以使用访问器从现有属性返回新的计算的值：

```php
/**
 * 获取用户的全名。
 *
 * @return string
 */
public function getFullNameAttribute()
{
    return "{$this->first_name} {$this->last_name}";
}
```

{% hint style="info" %}

如果你希望将这些计算值添加到你的模型的数组 / JSON 表示中，[你需要去追加它们](https://laravel.com/docs/5.8/eloquent-serialization#appending-values-to-json)。

{% endhint %}

### 定义一个存取器

要定义存取器，在你的模型上定义 `setFooAttribute` 方法，其中 `Foo` 是你要访问的列的『驼峰式』名称。所以，再次，让我们为 `first_name` 属性定义一个存取器。当我们尝试在模型上设置 `first_name` 属性的值时，将自动调用此存取器：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * 设置用户的名字。
     *
     * @param  string  $value
     * @return void
     */
    public function setFirstNameAttribute($value)
    {
        $this->attributes['first_name'] = strtolower($value);
    }
}
```

存取器将接收在属性上设置的值，允许你操作该值并在 Eloquent 模型的内部 `$attributes` 属性上设置操纵值。因此，例如，如果我们尝试将 `first_name` 的属性设置为 `Sally`：

```php
$user = App\User::find(1);

$user->first_name = 'Sally';
```

在此示例中，将使用值 `Sally` 时将调用 `setFirstNameAttribute` 方法。然后，存取器将 `strtolower` 函数应用于名称，并在内部 `$attributes` 数组中设置其结果值。

## 日期存取器

默认情况下，Eloquent 会将 `created_at` 和 `updated_at` 列转换为 [Carbon](https://github.com/briannesbitt/Carbon) 实例，这会扩展 PHP `DateTime` 类并提供各种有用的方法。你可以通过设置你的模型的 `$dates` 属性来添加其他日期属性：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * 应该转换为日期的属性。
     *
     * @var array
     */
    protected $dates = [
        'seen_at',
    ];
}
```

{% hint style="info" %}

你可以通过将你的模型的公共 `$timestamps` 属性设置为 `false` 来禁用默认的 `created_at` 和 `updated_at` 时间戳。

{% endhint %}
