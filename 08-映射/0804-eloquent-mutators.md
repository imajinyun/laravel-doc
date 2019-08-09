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

当列被视为日期时，你可以将其值设置为 UNIX 时间戳，日期字符串（`Y-m-d`），日期时间字符串或 `DateTime` / `Carbon` 实例。日期的值将被正确转换并存储在你的数据库中：

```php
$user = App\User::find(1);

$user->deleted_at = now();

$user->save();
```

***

如上所述，当检索在你的 `$dates` 属性中列出的属性时，它们将自动转换为 [Carbon](https://github.com/briannesbitt/Carbon) 实例，允许你在属性上使用 Carbon 的任何方法：

```php
$user = App\User::find(1);

return $user->deleted_at->getTimestamp();
```

***

**日期格式**

默认情况下，时间戳格式为 `'Y-m-d H:i:s'`。如果你需要自定义时间戳格式，在你的模型上设置 `$dateFormat` 属性。此属性确定数据库中日期属性的存储方式，以及将模型序列化为数组或 JSON 时的格式：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    /**
     * 模型日期列的存储格式。
     *
     * @var string
     */
    protected $dateFormat = 'U';
}
```

## 属性转换

在你的模型上的 `$cast` 属性提供了将属性转换为公共数据类型的方便方法。`$cast` 属性应该是一个数组，其中键是要转换的属性的名称，值是你希望将列转换为的类型。支持的转换类型有：`integer`、`real`、`float`、`double`、`decimal:<digits>`、`string`、`boolean`、`object`、`array`、`collection`、`date`、`datetime` 和 `timestamp`。当强制转换为 `decimal` 时，你必须定义精度位数（`decimal:2`）。

为了演示属性转换，让我们将 `is_admin` 属性转换为布尔值，该属性以整数（`0` 或 `1`）的形式存储在数据库中：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * 应该转换为原生类型的属性。
     *
     * @var array
     */
    protected $casts = [
        'is_admin' => 'boolean',
    ];
}
```

现在，当你访问 `is_admin` 属性时，它总是被转换为布尔值，即使底层值以整数的形式存储在数据库中：

```php
$user = App\User::find(1);

if ($user->is_admin) {
    //
}
```

### 数组 & JSON 转换

在处理存储为序列化 JSON 的列时，`array` 转换类型特别有用。例如，如果你的数据库具有包含序列化 JSON 的 `JSON` 或 `TEXT` 字段类型，则在你的Eloquent 模型上访问该属性时，添加到 `array` 转换的属性将自动将该属性反序列化为 PHP 数组：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * 应该转换为原生类型的属性。
     *
     * @var array
     */
    protected $casts = [
        'options' => 'array',
    ];
}
```

一旦定义了转换，你就可以访问 `options` 属性，它将自动从 JSON 反序列化为一个 PHP 数组。当你设置 `options` 属性的值时，给定的数组将自动序列化回 JSON 以供存储：

```php
$user = App\User::find(1);

$options = $user->options;

$options['key'] = 'value';

$user->options = $options;

$user->save();
```

### 日期转换

使用 `date` 或 `datetime` 转换类型时，你可以指定日期的格式。当 [模型序列化为数组或 JSON](https://laravel.com/docs/5.8/eloquent-serialization) 时，将使用此格式：

```php
/**
 * 应该转换为原生类型的属性。
 *
 * @var array
 */
protected $casts = [
    'created_at' => 'datetime:Y-m-d',
];
```
