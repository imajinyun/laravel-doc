# 助手

## 简介

Laravel 包含各种全局『助手』PHP 函数。其中许多函数由框架本身使用；但是，如果你觉得它们很方便，你可以在自己的应用程序中自由地使用它们。

## 可用方法

### 数组 & 对象列表

|                                                                             |                                                                           |                                                                                        |
| --------------------------------------------------------------------------- | ------------------------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| [Arr::add](https://laravel.com/docs/5.8/helpers#method-array-add)           | [Arr::has](https://laravel.com/docs/5.8/helpers#method-array-has)         | [Arr::sortRecursive](https://laravel.com/docs/5.8/helpers#method-array-sort-recursive) |
| [Arr::collapse](https://laravel.com/docs/5.8/helpers#method-array-collapse) | [Arr::last](https://laravel.com/docs/5.8/helpers#method-array-last)       | [Arr::where](https://laravel.com/docs/5.8/helpers#method-array-where)                  |
| [Arr::divide](https://laravel.com/docs/5.8/helpers#method-array-divide)     | [Arr::only](https://laravel.com/docs/5.8/helpers#method-array-only)       | [Arr::wrap](https://laravel.com/docs/5.8/helpers#method-array-wrap)                    |
| [Arr::dot](https://laravel.com/docs/5.8/helpers#method-array-dot)           | [Arr::pluck](https://laravel.com/docs/5.8/helpers#method-array-pluck)     | [data_fill](https://laravel.com/docs/5.8/helpers#method-data-fill)                     |
| [Arr::except](https://laravel.com/docs/5.8/helpers#method-array-except)     | [Arr::prepend](https://laravel.com/docs/5.8/helpers#method-array-prepend) | [data_get](https://laravel.com/docs/5.8/helpers#method-data-get)                       |
| [Arr::first](https://laravel.com/docs/5.8/helpers#method-array-first)       | [Arr::pull](https://laravel.com/docs/5.8/helpers#method-array-pull)       | [data_set](https://laravel.com/docs/5.8/helpers#method-data-set)                       |
| [Arr::flatten](https://laravel.com/docs/5.8/helpers#method-array-flatten)   | [Arr::random](https://laravel.com/docs/5.8/helpers#method-array-random)   | [head](https://laravel.com/docs/5.8/helpers#method-head)                               |
| [Arr::forget](https://laravel.com/docs/5.8/helpers#method-array-forget)     | [Arr::set](https://laravel.com/docs/5.8/helpers#method-array-set)         | [last](https://laravel.com/docs/5.8/helpers#method-last)                               |
| [Arr::get](https://laravel.com/docs/5.8/helpers#method-array-get)           | [Arr::sort](https://laravel.com/docs/5.8/helpers#method-array-sort)       |                                                                                        |

### 路径列表

|                                                                            |                                                                          |                                                                        |
| -------------------------------------------------------------------------- | ------------------------------------------------------------------------ | ---------------------------------------------------------------------- |
| [app_path](https://laravel.com/docs/5.8/helpers#method-app-path)           | [base_path](https://laravel.com/docs/5.8/helpers#method-base-path)       | [config_path](https://laravel.com/docs/5.8/helpers#method-config-path) |
| [database_path](https://laravel.com/docs/5.8/helpers#method-database-path) | [mix](https://laravel.com/docs/5.8/helpers#method-mix)                   | [public_path](https://laravel.com/docs/5.8/helpers#method-public-path) |
| [resource_path](https://laravel.com/docs/5.8/helpers#method-resource-path) | [storage_path](https://laravel.com/docs/5.8/helpers#method-storage-path) |

### 字符串列表

|                                                                                      |                                                                                    |                                                                            |
| ------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| [__](https://laravel.com/docs/5.8/helpers#method-__)                                 | [Str::finish](https://laravel.com/docs/5.8/helpers#method-str-finish)              | [Str::singular](https://laravel.com/docs/5.8/helpers#method-str-singular)  |
| [class_basename](https://laravel.com/docs/5.8/helpers#method-class-basename)         | [Str::is](https://laravel.com/docs/5.8/helpers#method-str-is)                      | [Str::slug](https://laravel.com/docs/5.8/helpers#method-str-slug)          |
| [e](https://laravel.com/docs/5.8/helpers#method-e)                                   | [Str::kebab](https://laravel.com/docs/5.8/helpers#method-kebab-case)               | [Str::snake](https://laravel.com/docs/5.8/helpers#method-snake-case)       |
| [preg_replace_array](https://laravel.com/docs/5.8/helpers#method-preg-replace-array) | [Str::limit](https://laravel.com/docs/5.8/helpers#method-str-limit)                | [Str::start](https://laravel.com/docs/5.8/helpers#method-str-start)        |
| [Str::after](https://laravel.com/docs/5.8/helpers#method-str-after)                  | [Str::orderedUuid](https://laravel.com/docs/5.8/helpers#method-str-ordered-uuid)   | [Str::startsWith](https://laravel.com/docs/5.8/helpers#method-starts-with) |
| [Str::before](https://laravel.com/docs/5.8/helpers#method-str-before)                | [Str::plural](https://laravel.com/docs/5.8/helpers#method-str-plural)              | [Str::studly](https://laravel.com/docs/5.8/helpers#method-studly-case)     |
| [Str::camel](https://laravel.com/docs/5.8/helpers#method-camel-case)                 | [Str::random](https://laravel.com/docs/5.8/helpers#method-str-random)              | [Str::title](https://laravel.com/docs/5.8/helpers#method-title-case)       |
| [Str::contains](https://laravel.com/docs/5.8/helpers#method-str-contains)            | [Str::replaceArray](https://laravel.com/docs/5.8/helpers#method-str-replace-array) | [Str::uuid](https://laravel.com/docs/5.8/helpers#method-str-uuid)          |
| [Str::containsAll](https://laravel.com/docs/5.8/helpers#method-str-contains-all)     | [Str::replaceFirst](https://laravel.com/docs/5.8/helpers#method-str-replace-first) | [trans](https://laravel.com/docs/5.8/helpers#method-trans)                 |
| [Str::endsWith](https://laravel.com/docs/5.8/helpers#method-ends-with)               | [Str::replaceLast](https://laravel.com/docs/5.8/helpers#method-str-replace-last)   | [trans_choice](https://laravel.com/docs/5.8/helpers#method-trans-choice)   |

### URL 列表

|                                                              |                                                                          |                                                                      |
| ------------------------------------------------------------ | ------------------------------------------------------------------------ | -------------------------------------------------------------------- |
| [action](https://laravel.com/docs/5.8/helpers#method-action) | [route](https://laravel.com/docs/5.8/helpers#method-route)               | [secure_url](https://laravel.com/docs/5.8/helpers#method-secure-url) |
| [asset](https://laravel.com/docs/5.8/helpers#method-asset)   | [secure_asset](https://laravel.com/docs/5.8/helpers#method-secure-asset) | [url](https://laravel.com/docs/5.8/helpers#method-url)               |

### 杂项列表

|                                                                                          |                                                                          |                                                                          |
| ---------------------------------------------------------------------------------------- | ------------------------------------------------------------------------ | ------------------------------------------------------------------------ |
| [abort](https://laravel.com/docs/5.8/helpers#method-abort)                               | [decrypt](https://laravel.com/docs/5.8/helpers#method-decrypt)           | [report](https://laravel.com/docs/5.8/helpers#method-report)             |
| [abort_if](https://laravel.com/docs/5.8/helpers#method-abort-if)                         | [dispatch](https://laravel.com/docs/5.8/helpers#method-dispatch)         | [request](https://laravel.com/docs/5.8/helpers#method-request)           |
| [abort_unless](https://laravel.com/docs/5.8/helpers#method-abort-unless)                 | [dispatch_now](https://laravel.com/docs/5.8/helpers#method-dispatch-now) | [rescue](https://laravel.com/docs/5.8/helpers#method-rescue)             |
| [app](https://laravel.com/docs/5.8/helpers#method-app)                                   | [dump](https://laravel.com/docs/5.8/helpers#method-dump)                 | [resolve](https://laravel.com/docs/5.8/helpers#method-resolve)           |
| [auth](https://laravel.com/docs/5.8/helpers#method-auth)                                 | [encrypt](https://laravel.com/docs/5.8/helpers#method-encrypt)           | [response](https://laravel.com/docs/5.8/helpers#method-response)         |
| [back](https://laravel.com/docs/5.8/helpers#method-back)                                 | [env](https://laravel.com/docs/5.8/helpers#method-env)                   | [retry](https://laravel.com/docs/5.8/helpers#method-retry)               |
| [bcrypt](https://laravel.com/docs/5.8/helpers#method-bcrypt)                             | [event](https://laravel.com/docs/5.8/helpers#method-event)               | [session](https://laravel.com/docs/5.8/helpers#method-session)           |
| [blank](https://laravel.com/docs/5.8/helpers#method-blank)                               | [factory](https://laravel.com/docs/5.8/helpers#method-factory)           | [tap](https://laravel.com/docs/5.8/helpers#method-tap)                   |
| [broadcast](https://laravel.com/docs/5.8/helpers#method-broadcast)                       | [filled](https://laravel.com/docs/5.8/helpers#method-filled)             | [throw_if](https://laravel.com/docs/5.8/helpers#method-throw-if)         |
| [cache](https://laravel.com/docs/5.8/helpers#method-cache)                               | [info](https://laravel.com/docs/5.8/helpers#method-info)                 | [throw_unless](https://laravel.com/docs/5.8/helpers#method-throw-unless) |
| [class_uses_recursive](https://laravel.com/docs/5.8/helpers#method-trait-uses-recursive) | [logger](https://laravel.com/docs/5.8/helpers#method-logger)             | [today](https://laravel.com/docs/5.8/helpers#method-today)               |
| [collect](https://laravel.com/docs/5.8/helpers#method-collect)                           | [method_field](https://laravel.com/docs/5.8/helpers#method-method-field) | [trait_uses_recursive](https://laravel.com/docs/5.8/helpers#method-env)  |
| [config](https://laravel.com/docs/5.8/helpers#method-config)                             | [now](https://laravel.com/docs/5.8/helpers#method-now)                   | [transform](https://laravel.com/docs/5.8/helpers#method-transform)       |
| [cookie](https://laravel.com/docs/5.8/helpers#method-cookie)                             | [old](https://laravel.com/docs/5.8/helpers#method-old)                   | [validator](https://laravel.com/docs/5.8/helpers#method-validator)       |
| [csrf_field](https://laravel.com/docs/5.8/helpers#method-csrf-field)                     | [optional](https://laravel.com/docs/5.8/helpers#method-optional)         | [value](https://laravel.com/docs/5.8/helpers#method-value)               |
| [csrf_token](https://laravel.com/docs/5.8/helpers#method-csrf-token)                     | [policy](https://laravel.com/docs/5.8/helpers#method-policy)             | [view](https://laravel.com/docs/5.8/helpers#method-view)                 |
| [dd](https://laravel.com/docs/5.8/helpers#method-dd)                                     | [redirect](https://laravel.com/docs/5.8/helpers#method-redirect)         | [with](https://laravel.com/docs/5.8/helpers#method-with)                 |

## 方法列表

## 数组 & 对象

### `Arr::add()`

如果给定的键在数组中不存在或设置为 `null`，则 `Arr::add` 方法将给定的键 / 值对添加到数组：

```php
use Illuminate\Support\Arr;

$array = Arr::add(['name' => 'Desk'], 'price', 100);

// ['name' => 'Desk', 'price' => 100]

$array = Arr::add(['name' => 'Desk', 'price' => null], 'price', 100);

// ['name' => 'Desk', 'price' => 100]
```

### `Arr::collapse()`

`Arr::collapse` 方法将数组折叠为单个数组：

```php
use Illuminate\Support\Arr;

$array = Arr::collapse([[1, 2, 3], [4, 5, 6], [7, 8, 9]]);

// [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

### `Arr::divide()`

`Arr::divide` 方法返回两个数组，一个包含键，另一个包含给定数组的值：

```php
use Illuminate\Support\Arr;

[$keys, $values] = Arr::divide(['name' => 'Desk']);

// $keys: ['name']

// $values: ['Desk']
```

### `Arr::dot()`

`Arr::dot` 方法将多维数组扁平化为使用『点』表示深度的单层数组：

```php
use Illuminate\Support\Arr;

$array = ['products' => ['desk' => ['price' => 100]]];

$flattened = Arr::dot($array);

// ['products.desk.price' => 100]
```

### `Arr::except()`

`Arr::except` 从数组中删除给定的键 / 值对：

```php
use Illuminate\Support\Arr;

$array = ['name' => 'Desk', 'price' => 100];

$filtered = Arr::except($array, ['price']);

// ['name' => 'Desk']
```

### `Arr::first()`

`Arr::first` 方法返回通过给定真值测试的数组的第一个元素：

```php
use Illuminate\Support\Arr;

$array = [100, 200, 300];

$first = Arr::first($array, function ($value, $key) {
    return $value >= 150;
});

// 200
```

默认值也可以作为第三个参数传递给方法。如果没有值通过真值测试，则返回此值：

```php
use Illuminate\Support\Arr;

$first = Arr::first($array, $callback, $default);
```

### `Arr::flatten()`

`Arr::flatten` 方法将多维数组扁平化为一个单层数组：

```php
use Illuminate\Support\Arr;

$array = ['name' => 'Joe', 'languages' => ['PHP', 'Ruby']];

$flattened = Arr::flatten($array);

// ['Joe', 'PHP', 'Ruby']
```

### `Arr::forget()`

`Arr::forget` 方法使用『点』表示法从深度嵌套的数组中删除给定的键 / 值对：

```php
use Illuminate\Support\Arr;

$array = ['products' => ['desk' => ['price' => 100]]];

Arr::forget($array, 'products.desk');

// ['products' => []]
```

### `Arr::get()`

`Arr::get` 方法使用『点』表示法从深层嵌套数组中检索值：

```php
use Illuminate\Support\Arr;

$array = ['products' => ['desk' => ['price' => 100]]];

$price = Arr::get($array, 'products.desk.price');

// 100
```

`Arr::get` 方法还接受一个默认值，如果没有找到特定的键，将返回该值：

```php
use Illuminate\Support\Arr;

$discount = Arr::get($array, 'products.desk.discount', 0);

// 0
```

### `Arr::has()`

`Arr::has` 方法使用『点』符号检查数组中是否存在给定条目或条目：

```php
use Illuminate\Support\Arr;

$array = ['product' => ['name' => 'Desk', 'price' => 100]];

$contains = Arr::has($array, 'product.name');

// true

$contains = Arr::has($array, ['product.price', 'product.discount']);

// false
```

### `Arr::last()`

`Arr::last` 方法返回通过给定真值测试的数组的最后一个元素：

```php
use Illuminate\Support\Arr;

$array = [100, 200, 300, 110];

$last = Arr::last($array, function ($value, $key) {
    return $value >= 150;
});

// 300
```

可以将默认值作为第三个参数传递给方法。如果没有值通过真值测试，则返回此值：

```php
use Illuminate\Support\Arr;

$last = Arr::last($array, $callback, $default);
```

### `Arr::only()`

`Arr::only` 方法仅返回从给定数组中指定的键 / 值对：

```php
use Illuminate\Support\Arr;

$array = ['name' => 'Desk', 'price' => 100, 'orders' => 10];

$slice = Arr::only($array, ['name', 'price']);

// ['name' => 'Desk', 'price' => 100]
```

### `Arr::pluck()`

`Arr::pluck` 方法从数组中检索给定键的所有值：

```php
use Illuminate\Support\Arr;

$array = [
    ['developer' => ['id' => 1, 'name' => 'Taylor']],
    ['developer' => ['id' => 2, 'name' => 'Abigail']],
];

$names = Arr::pluck($array, 'developer.name');

// ['Taylor', 'Abigail']
```

你还可以指定希望如何键入结果列表：

```php
use Illuminate\Support\Arr;

$names = Arr::pluck($array, 'developer.name', 'developer.id');

// [1 => 'Taylor', 2 => 'Abigail']
```

### `Arr::prepend()`

`Arr::prepend` 方法将把一个条目推到数组的开头：

```php
use Illuminate\Support\Arr;

$array = ['one', 'two', 'three', 'four'];

$array = Arr::prepend($array, 'zero');

// ['zero', 'one', 'two', 'three', 'four']
```

如果需要，你可以指定值应该使用的键：

```php
use Illuminate\Support\Arr;

$array = ['price' => 100];

$array = Arr::prepend($array, 'Desk', 'name');

// ['name' => 'Desk', 'price' => 100]
```

### `Arr::pull()`

`Arr::pull` 方法返回并从数组中删除的键 / 值对：

```php
use Illuminate\Support\Arr;

$array = ['name' => 'Desk', 'price' => 100];

$name = Arr::pull($array, 'name');

// $name: Desk

// $array: ['price' => 100]
```

可以将默认值作为第三个参数传递给方法。如果键不存在，则返回此值：

```php
use Illuminate\Support\Arr;

$value = Arr::pull($array, $key, $default);
```

### `Arr::random()`

`Arr::random` 方法从数组中返回一个随机值：

```php
use Illuminate\Support\Arr;

$array = [1, 2, 3, 4, 5];

$random = Arr::random($array);

// 4 - (retrieved randomly)
```

你还可以指定要返回的条目数作为可选的第二个参数。注意，提供这个参数将返回一个数组，即使只期望一个条目：

```php
use Illuminate\Support\Arr;

$items = Arr::random($array, 2);

// [2, 5] - (retrieved randomly)
```

### `Arr::set()`

`Arr::set` 方法使用『点』符号在深度嵌套数组中设置值：

```php
use Illuminate\Support\Arr;

$array = ['products' => ['desk' => ['price' => 100]]];

Arr::set($array, 'products.desk.price', 200);

// ['products' => ['desk' => ['price' => 200]]]
```

### `Arr::sort()`

`Arr::sort` 方法按其值对数组进行排序：

```php
use Illuminate\Support\Arr;

$array = ['Desk', 'Table', 'Chair'];

$sorted = Arr::sort($array);

// ['Chair', 'Desk', 'Table']
```

你还可以根据给定闭包的结果对数组进行排序：

```php
use Illuminate\Support\Arr;

$array = [
    ['name' => 'Desk'],
    ['name' => 'Table'],
    ['name' => 'Chair'],
];

$sorted = array_values(Arr::sort($array, function ($value) {
    return $value['name'];
}));

/*
    [
        ['name' => 'Chair'],
        ['name' => 'Desk'],
        ['name' => 'Table'],
    ]
*/
```

### `Arr::sortRecursive()`

`Arr::sortRecursive` 方法使用 `sort` 函数对数值子数组进行递归排序，并对关联子数组使用 `ksort` 函数排序：

```php
use Illuminate\Support\Arr;

$array = [
    ['Roman', 'Taylor', 'Li'],
    ['PHP', 'Ruby', 'JavaScript'],
    ['one' => 1, 'two' => 2, 'three' => 3],
];

$sorted = Arr::sortRecursive($array);

/*
    [
        ['JavaScript', 'PHP', 'Ruby'],
        ['one' => 1, 'three' => 3, 'two' => 2],
        ['Li', 'Roman', 'Taylor'],
    ]
*/
```

### `Arr::where()`

`Arr::where` 方法使用给定的闭包过滤数组：

```php
use Illuminate\Support\Arr;

$array = [100, '200', 300, '400', 500];

$filtered = Arr::where($array, function ($value, $key) {
    return is_string($value);
});

// [1 => '200', 3 => '400']
```

### `Arr::wrap()`

`Arr::wrap` 方法将给定的值包装在数组中。如果给定值已经是数组，则不会更改它：

```php
use Illuminate\Support\Arr;

$string = 'Laravel';

$array = Arr::wrap($string);

// ['Laravel']
```

如果给定值为空，则返回一个空数组：

```php
use Illuminate\Support\Arr;

$nothing = null;

$array = Arr::wrap($nothing);

// []
```

### `data_fill()`

`data_fill` 数据填充函数使用『点』符号在嵌套数组或对象中设置缺失的值：

```php
$data = ['products' => ['desk' => ['price' => 100]]];

data_fill($data, 'products.desk.price', 200);

// ['products' => ['desk' => ['price' => 100]]]

data_fill($data, 'products.desk.discount', 10);

// ['products' => ['desk' => ['price' => 100, 'discount' => 10]]]
```

该函数还接受星号作为通配符，并将相应地填充目标：

```php
$data = [
    'products' => [
        ['name' => 'Desk 1', 'price' => 100],
        ['name' => 'Desk 2'],
    ],
];

data_fill($data, 'products.*.price', 200);

/*
    [
        'products' => [
            ['name' => 'Desk 1', 'price' => 100],
            ['name' => 'Desk 2', 'price' => 200],
        ],
    ]
*/
```

### `data_get()`

`data_get` 函数使用『点』符号从嵌套数组或对象中检索值：

```php
$data = ['products' => ['desk' => ['price' => 100]]];

$price = data_get($data, 'products.desk.price');

// 100
```

`data_get` 函数还接受一个默认值，如果没有找到指定的键，则返回该值：

```php
$discount = data_get($data, 'products.desk.discount', 0);

// 0
```

该函数还使用星号接受通配符，星号可以针对数组或对象的任何键：

```php
$data = [
    'product-one' => ['name' => 'Desk 1', 'price' => 100],
    'product-two' => ['name' => 'Desk 2', 'price' => 150],
];

data_get($data, '*.name');

// ['Desk 1', 'Desk 2'];
```

### `data_set`

`data_set` 函数使用『点』符号在嵌套数组或对象中设置值：

```php
$data = ['products' => ['desk' => ['price' => 100]]];

data_set($data, 'products.desk.price', 200);

// ['products' => ['desk' => ['price' => 200]]]
```

该函数还接受通配符，并将相应地在目标上设置值：

```php
$data = [
    'products' => [
        ['name' => 'Desk 1', 'price' => 100],
        ['name' => 'Desk 2', 'price' => 150],
    ],
];

data_set($data, 'products.*.price', 200);

/*
    [
        'products' => [
            ['name' => 'Desk 1', 'price' => 200],
            ['name' => 'Desk 2', 'price' => 200],
        ],
    ]
*/
```

默认情况下，将覆盖任何存在的值。如果你只想设置一个值，如果它不存在，你可以传递 `false` 作为第四个参数：

```php
$data = ['products' => ['desk' => ['price' => 100]]];

data_set($data, 'products.desk.price', 200, false);

// ['products' => ['desk' => ['price' => 100]]]
```

### `head()`

`head` 函数返回给定数组中的第一个元素：

```php
$array = [100, 200, 300];

$first = head($array);

// 100
```

### `last()`

`last` 函数返回给定数组中的最后一个元素：

```php
$array = [100, 200, 300];

$last = last($array);

// 300
```

## 路径

### `app_path()`

`app_path` 函数返回 `app` 目录的完全限定路径。你还可以使用 `app_path` 函数生成相对于应用程序目录的文件的完全限定路径：

```php
$path = app_path();

$path = app_path('Http/Controllers/Controller.php');
```

### `base_path()`

`base_path` 函数返回到项目根的完全限定路径。你还可以使用 `base_path` 函数生成相对于项目根目录的给定文件的完全限定路径：

```php
$path = base_path();

$path = base_path('vendor/bin');
```

### `config_path()`

`config_path` 函数返回 `config` 目录的完全限定路径。你还可以使用 `config_path` 函数生成应用程序配置目录中给定文件的完全限定路径：

```php
$path = config_path();

$path = config_path('app.php');
```

### `database_path()`

`database_path` 函数返回 `database` 目录的完全限定路径。你还可以使用 `database_path` 函数生成数据库目录中给定文件的完全限定路径：

```php
$path = database_path();

$path = database_path('factories/UserFactory.php');
```

### `mix()`

`mix` 函数返回 [版本化的 Mix 文件](https://laravel.com/docs/5.8/mix) 的路径：

```php
$path = mix('css/app.css');
```

### `public_path()`

`public_path` 函数返回 `public` 目录的完全限定路径。你还可以使用 `public_path` 函数生成公共目录中给定文件的完全限定路径：

```php
$path = public_path();

$path = public_path('css/app.css');
```

### `resource_path()`

`resource_path` 函数返回 `resources` 目录的完全限定路径。你还可以使用 `resource_path` 函数生成资源目录中给定文件的完全限定路径：

```php
$path = resource_path();

$path = resource_path('sass/app.scss');
```

### `storage_path()`

`storage_path` 函数返回 `storage` 目录的完全限定路径。你还可以使用 `storage_path` 函数生成存储目录中给定文件的完全限定路径：

```php
$path = storage_path();

$path = storage_path('app/file.txt');
```

## 字符串

### `__()`

`__` 函数使用你的 [本地化文件](https://laravel.com/docs/5.8/localization) 翻译给定的翻译字符串或翻译键：

```php
echo __('Welcome to our application');

echo __('messages.welcome');
```

如果指定的翻译字符串或键不存在，`__` 函数将返回给定的值。因此，使用上面的示例，如果翻译键不存在，`__` 函数将返回 `messages.welcome`。

### `class_basename()`

`class_basename` 函数返回给定类的类名，其中类的命名空间已被移除：

```php
$class = class_basename('Foo\Bar\Baz');

// Baz
```

### `e()`

`e` 函数运行 PHP 的 `htmlspecialchars` 函数，默认情况下，`double_encode` 选项设置为 `true`：

```php
echo e('<html>foo</html>');

// &lt;html&gt;foo&lt;/html&gt;
```

### `preg_replace_array()`

`preg_replace_array` 函数使用数组依次替换字符串中的给定模式：

```php
$string = 'The event will take place between :start and :end';

$replaced = preg_replace_array('/:[a-z_]+/', ['8:30', '9:00'], $string);

// The event will take place between 8:30 and 9:00
```

### `Str::after()`

`Str::after` 方法返回字符串中给定值之后的所有内容：

```php
use Illuminate\Support\Str;

$slice = Str::after('This is my name', 'This is');

// ' my name'
```

### `Str::before()`

`Str::before` 方法方法返回在字符串给定值之前的所有内容：

```php
use Illuminate\Support\Str;

$slice = Str::before('This is my name', 'my name');

// 'This is '
```

### `Str::camel()`

`Str::camel` 方法转换给定的字符串到 `camelCase`：

```php
use Illuminate\Support\Str;

$converted = Str::camel('foo_bar');

// fooBar
```

### `Str::contains()`

`Str::contains` 方法确定给定字符串是否包含给定值（区分大小写）：

```php
use Illuminate\Support\Str;

$contains = Str::contains('This is my name', 'my');

// true
```

你还可以传递一个值数组，以确定给定字符串是否包含任何值：

```php
use Illuminate\Support\Str;

$contains = Str::contains('This is my name', ['my', 'foo']);

// true
```

### `Str::containsAll()`

`Str::containsAll` 方法确定给定字符串是否包含所有数组值：

```php
use Illuminate\Support\Str;

$containsAll = Str::containsAll('This is my name', ['my', 'name']);

// true
```

### `Str::endsWith()`

`Str::endsWith` 方法确定给定字符串是否以给定值结束：

```php
use Illuminate\Support\Str;

$result = Str::endsWith('This is my name', 'name');

// true
```

### `Str::finish()`

`Str::finish` 方法将给定值的一个实例添加到一个字符串中，如果它还没有以该值结束：

```php
use Illuminate\Support\Str;

$adjusted = Str::finish('this/string', '/');

// this/string/

$adjusted = Str::finish('this/string/', '/');

// this/string/
```

### `Str::is()`

`Str::is` 方法确定给定字符串是否与给定模式匹配。星号可以用来表示通配符：

```php
use Illuminate\Support\Str;

$matches = Str::is('foo*', 'foobar');

// true

$matches = Str::is('baz*', 'foobar');

// false
```

### `Str::kebab()`

`Str::kebab` 转换给定的字符串为 `kebab-case`：

```php
use Illuminate\Support\Str;

$converted = Str::kebab('fooBar');

// foo-bar
```

### Str::limit()

`Str::limit` 方法以指定的长度截断给定的字符串：

```php
use Illuminate\Support\Str;

$truncated = Str::limit('The quick brown fox jumps over the lazy dog', 20);

// The quick brown fox...
```

你还可以传递第三个参数来更改将附加到末尾的字符串：

```php
use Illuminate\Support\Str;

$truncated = Str::limit('The quick brown fox jumps over the lazy dog', 20, ' (...)');

// The quick brown fox (...)
```

### `Str::orderedUuid()`

`Str::orderedUuid` 方法生成一个『时间戳优先的』UUID，它可以有效地存储在索引数据库列中：

```php
use Illuminate\Support\Str;

return (string) Str::orderedUuid();
```

### `Str::plural()`

`Str::plural` 方法将字符串转换为其复数形式。这个函数目前只支持英语：

```php
use Illuminate\Support\Str;

$plural = Str::plural('car');

// cars

$plural = Str::plural('child');

// children
```

你可以提供一个整数作为函数的第二个参数来检索字符串的单数或复数形式：

```php
use Illuminate\Support\Str;

$plural = Str::plural('child', 2);

// children

$plural = Str::plural('child', 1);

// child
```

### `Str::random()`

`Str::random` 方法生成指定长度的随机字符串。这个函数使用 PHP 的 `random_bytes` 函数：

```php
use Illuminate\Support\Str;

$random = Str::random(40);
```

### `Str::replaceArray()`

`Str::replaceArray` 方法使用数组顺序替换字符串中的给定值：

```php
use Illuminate\Support\Str;

$string = 'The event will take place between ? and ?';

$replaced = Str::replaceArray('?', ['8:30', '9:00'], $string);

// The event will take place between 8:30 and 9:00
```

### `Str::replaceFirst()`

`Str::replaceFirst` 方法替换字符串中给定值的第一次出现：

```php
use Illuminate\Support\Str;

$replaced = Str::replaceFirst('the', 'a', 'the quick brown fox jumps over the lazy dog');

// a quick brown fox jumps over the lazy dog
```

### `Str::replaceLast()`

`Str::replaceLast` 方法替换字符串中给定值的最后一次出现：

```php
use Illuminate\Support\Str;

$replaced = Str::replaceLast('the', 'a', 'the quick brown fox jumps over the lazy dog');

// the quick brown fox jumps over a lazy dog
```

### `Str::singular()`

`Str::singular` 方法将字符串转换为它的单数形式。这个函数目前只支持英语：

```php
use Illuminate\Support\Str;

$singular = Str::singular('cars');

// car

$singular = Str::singular('children');

// child
```

### `Str::slug()`

`Str::slug` 方法从给定的字符串生成一个 URL 友好的 `slug`：

```php
use Illuminate\Support\Str;

$slug = Str::slug('Laravel 5 Framework', '-');

// laravel-5-framework
```

### `Str::snake()`

`Str::snake` 方法将给定的字符串转换为 `snake_case`：

```php
use Illuminate\Support\Str;

$converted = Str::snake('fooBar');

// foo_bar
```

### `Str::start()`

`Str::start` 方法将给定值的单个实例添加到字符串中，如果字符串还没有以该值开始：

```php
use Illuminate\Support\Str;

$adjusted = Str::start('this/string', '/');

// /this/string

$adjusted = Str::start('/this/string', '/');

// /this/string
```

### `Str::startsWith()`

`Str::startsWith` 方法确定给定字符串是否以给定值开头：

```php
use Illuminate\Support\Str;

$result = Str::startsWith('This is my name', 'This');

// true
```

### `Str::studly()`

`Str::studly` 方法将给定的字符串转换为 `StudlyCase`：

```php
use Illuminate\Support\Str;

$converted = Str::studly('foo_bar');

// FooBar
```

### `Str::title()`

`Str::title` 方法将给定的字符串转换为 `Title Case`：

```php
use Illuminate\Support\Str;

$converted = Str::title('a nice title uses the correct case');

// A Nice Title Uses The Correct Case
```

### `Str::uuid()`

`Str::uuid` 方法生成一个 UUID（版本 4）：

```php
use Illuminate\Support\Str;

return (string) Str::uuid();
```

### `trans()`

`trans` 函数使用 [本地化文件](https://laravel.com/docs/5.8/localization) 翻译给定的翻译键：

```php
echo trans('messages.welcome');
```

如果指定的转换键不存在，则 `trans` 函数将返回给定的键。因此，使用上面的示例，如果转换键不存在，`trans` 函数将返回 `messages.welcome`。

### `trans_choice()`

`trans_choice` 函数用词形转换给定的翻译键：

```php
echo trans_choice('messages.notifications', $unreadCount);
```

如果指定的翻译键不存在，`trans_choice` 函数将返回给定的键。因此，使用上面的示例，如果翻译键不存在，则 `trans_choice` 函数将返回 `messages.notifications`。

## URL

### `action()`

`action` 函数为给定的控制器动作生成一个 URL。你不需要传递控制器的完整名称空间。相反，传递相对于 `App\Http\Controllers` 命名空间的控制器类名：

```php
$url = action('HomeController@index');

$url = action([HomeController::class, 'index']);
```

如果方法接受路由参数，可以将它们作为第二个参数传递给方法：

```php
$url = action('UserController@profile', ['id' => 1]);
```

### `asset()`

`asset` 函数使用请求的当前模式（HTTP 或 HTTPS）为资产生成 URL：

```php
$url = asset('img/photo.jpg');
```

你可以通过在 `.env` 文件中设置 `ASSET_URL` 变量来配置资产 URL 主机。如果你将资产托管在 Amazon S3 这样的外部服务上，这将非常有用：

```php
// ASSET_URL=http://example.com/assets

$url = asset('img/photo.jpg'); // http://example.com/assets/img/photo.jpg
```

### `route()`

`route` 函数为给定的命名路由生成 URL：

```php
$url = route('routeName');
```

如果路由接受参数，可以将它们作为第二个参数传递给方法：

```php
$url = route('routeName', ['id' => 1]);
```

默认情况下，`route` 函数生成一个绝对 URL。如果你希望生成相对 URL，可以将 `false` 作为第三个参数传递：

```php
$url = route('routeName', ['id' => 1], false);
```

### `secure_asset()`

`secure_asset` 函数使用 HTTPS 为资产生成 URL：

```php
$url = secure_asset('img/photo.jpg');
```

### `secure_url()`

`secure_url` 函数为给定路径生成完全限定的 HTTPS URL：

```php
$url = secure_url('user/profile');

$url = secure_url('user/profile', [1]);
```

### `url()`

`url` 函数为给定路径生成一个完全限定的 URL：

```php
$url = url('user/profile');

$url = url('user/profile', [1]);
```

如果路径没有提供，将返回一个 `Illuminate\Routing\UrlGenerator` 实例：

```php
$current = url()->current();

$full = url()->full();

$previous = url()->previous();
```

## 杂项

### `abort()`

`abort` 函数抛出 [一个 HTTP 异常](https://laravel.com/docs/5.8/errors#http-exceptions)，该异常将由 [异常处理程序](https://laravel.com/docs/5.8/errors#the-exception-handler) 渲染：

```php
abort(403);
```

你还可以提供异常的响应文本和自定义响应头：

```php
abort(403, 'Unauthorized.', $headers);
```

### `abort_if()`

如果给定的布尔表达式计算结果为 `true`，`abort_if` 函数将抛出 HTTP 异常：

```php
abort_if(! Auth::user()->isAdmin(), 403);
```

与 `abort` 方法类似，你还可以将异常的响应文本作为第三个参数，并将自定义响应头数组作为第四个参数。

### `abort_unless()`

如果给定的布尔表达式计算结果为 `false`，`abort_unless` 函数抛出 HTTP 异常：

```php
abort_unless(Auth::user()->isAdmin(), 403);
```

与 `abort` 方法类似，你还可以将异常的响应文本作为第三个参数，并将自定义响应头数组作为第四个参数。

### `app()`

`app` 函数返回 [服务容器](https://laravel.com/docs/5.8/container) 实例：

```php
$container = app();
```

你可以传递一个类名或接口名来从容器中解析它：

```php
$api = app('HelpSpot\API');
```

### `auth()`

`auth` 函数返回一个 [认证者](https://laravel.com/docs/5.8/authentication) 实例。为了方便，你可以使用它代替 `Auth` 外观：

```php
$user = auth()->user();
```

如果需要，可以指定要访问哪个守卫实例：

```php
$user = auth('admin')->user();
```

### `back()`

`back` 函数生成一个 [重定向 HTTP 响应](https://laravel.com/docs/5.8/responses#redirects) 到用户之前的位置：

```php
return back($status = 302, $headers = [], $fallback = false);

return back();
```

### `bcrypt()`

`bcrypt` 函数使用 Bcrypt [散列](https://laravel.com/docs/5.8/hashing) 给定的值。你可以使用它作为 `Hash` 外观的替代：

```php
$password = bcrypt('my-secret-password');
```

### `blank()`

`blank` 函数返回给定值是否为『空』：

```phpblank('');
blank('   ');
blank(null);
blank(collect());

// true

blank(0);
blank(true);
blank(false);

// false
```

有关 `blank` 的逆函数，请参阅 [填充](https://laravel.com/docs/5.8/helpers#method-filled) 方法。

### `broadcast()`

`broadcast` 函数 [广播](https://laravel.com/docs/5.8/broadcasting) 给定的 [事件](https://laravel.com/docs/5.8/events) 给它的侦听器：

```php
broadcast(new UserRegistered($user));
```

### `cache()`

`cache` 函数可用于从 [缓存](https://laravel.com/docs/5.8/cache) 中获取值。如果给定的键在缓存中不存在，将返回一个可选的默认值：

```php
$value = cache('key');

$value = cache('key', 'default');
```

你可以通过将键 / 值对数组传递给函数来将条目添加到缓存中。你还应当传递缓存值应被视为有效的秒数或持续时间：

```php
cache(['key' => 'value'], 300);

cache(['key' => 'value'], now()->addSeconds(10));
```

### `class_uses_recursive()`

`class_uses_recursive` 函数返回一个类使用的所有特征，包括其所有父类使用的特征：

```php
$traits = class_uses_recursive(App\User::class);
```

### `collect()`

`collect` 函数从给定值创建一个 [集合](https://laravel.com/docs/5.8/collections) 实例：

```php
$collection = collect(['taylor', 'abigail']);
```

### `config()`

`config` 获取 [配置](https://laravel.com/docs/5.8/configuration) 变量的值。可以使用『点』语法访问配置值，其中包括文件的名称和希望访问的选项。可以指定默认值，如果不存在配置选项，则返回默认值：

```php
$value = config('app.timezone');

$value = config('app.timezone', $default);
```

你可以通过传递键 / 值对数组在运行时设置配置变量：

```php
config(['app.debug' => true]);
```

### `cookie()`

`cookie` 函数创建一个新的 [Cookie](https://laravel.com/docs/5.8/requests#cookies) 实例：

```php
$cookie = cookie('name', 'value', $minutes);
```

### `csrf_field()`

`csrf_field` 函数生成一个包含 CSRF 令牌值的 HTML `hidden` 输入字段。例如，使用 [Blade 语法](https://laravel.com/docs/5.8/blade)：

```php
{{ csrf_field() }}
```

### `csrf_token()`

`csrf_token` 函数检索当前 CSRF 令牌的值：

```php
$token = csrf_token();
```

### `dd()`

`dd` 函数转储给定的变量并结束脚本的执行：

```php
dd($value);

dd($value1, $value2, $value3, ...);
```

如果不想停止脚本的执行，可以使用 [dump](https://laravel.com/docs/5.8/helpers#method-dump) 函数。

### `decrypt()`

`decrypt` 函数使用 Laravel 的 [加密器](https://laravel.com/docs/5.8/encryption) 解密给定值：

```php
$decrypted = decrypt($encrypted_value);
```

### `dispatch()`

`dispatch` 函数将给定的 [作业](https://laravel.com/docs/5.8/queues#creating-jobs) 推到 Laravel [作业队列](https://laravel.com/docs/5.8/queues)：

```php
dispatch(new App\Jobs\SendEmails);
```

### `dispatch_now()`

`dispatch_now` 函数立即运行给定的 [作业](https://laravel.com/docs/5.8/queues#creating-jobs)，并从其 `handle` 方法返回值：

```php
$result = dispatch_now(new App\Jobs\SendEmails);
```

### `dump()`

`dump` 函数转储给定的变量：

```php
dump($value);

dump($value1, $value2, $value3, ...);
```

如果你想在转储变量之后停止执行脚本，请使用 [dd](https://laravel.com/docs/5.8/helpers#method-dd) 函数替代。

{% hint style="info" %}

你可以使用 Artisan 的 `dump-server` 命令拦截所有 `dump` 调用，并将它们显示在控制台窗口而不是浏览器中。

{% endhint %}

### `encrypt()`

`encrypt` 函数使用 Laravel 的 [加密器](https://laravel.com/docs/5.8/encryption) 加密给定的值：

```php
$encrypted = encrypt($unencrypted_value);
```

### `env()`

`env` 函数检索 [环境变量](https://laravel.com/docs/5.8/configuration#environment-configuration) 的值或返回默认值：

```php
$env = env('APP_ENV');

// 如果 APP_ENV 没有设置，返回 'production'...
$env = env('APP_ENV', 'production');
```

{% hint style="dagner" %}

如果在部署过程中执行 `config:cache` 命令，应该确保只从配置文件中调用 `env` 函数。一旦配置被缓存，`.env` 文件将不会被加载，所有对 `env` 函数的调用都将返回 `null`。

{% endhint %}

### `event()`

`event` 函数将给定的事件调度给它的 [侦听器](https://laravel.com/docs/5.8/events)：

```php
event(new UserRegistered($user));
```

### `factory()`

`factory` 函数为给定的类、名称和数量创建模型工厂生成器。它可以在 [测试](https://laravel.com/docs/5.8/database-testing#writing-factories) 或 [播种](https://laravel.com/docs/5.8/seeding#using-model-factories) 时使用：

```php
$user = factory(App\User::class)->make();
```

### `filled()`

`filled` 函数返回给定值是否为不『空』：

```php
filled(0);
filled(true);
filled(false);

// true

filled('');
filled('   ');
filled(null);
filled(collect());

// false
```

有关 `filled` 的逆函数，请参 [空](https://laravel.com/docs/5.8/helpers#method-blank) 函数。

### `info()`

`info` 函数将把信息写入日志：

```php
info('Some helpful information!');
```

上下文数据数组也可以传递给函数：

```php
info('User login attempt failed.', ['id' => $user->id]);
```

### `logger()`

`logger` 函数可用于向 [日志](https://laravel.com/docs/5.8/logging) 写入 `debug` 级别的消息：

```php
logger('Debug message');
```

上下文数据数组也可以传递给函数：

```php
logger('User has logged in.', ['id' => $user->id]);
```

如果没有向函数传递任何值，则返回一个 [logger](https://laravel.com/docs/5.8/errors#logging) 实例：

```php
logger()->error('You are not allowed here.');
```

### `method_field()`

`method_field` 函数生成一个 HTML `hidden` 输入字段，其中包含表单的 HTTP 动词欺骗值。例如，使用 [Blade 语法](https://laravel.com/docs/5.8/blade)：

```php
<form method="POST">
    {{ method_field('DELETE') }}
</form>
```

### `now()`

`now` 函数为当前时间创建一个新的 `Illuminate\Support\Carbon` 实例：

```php
$now = now();
```

### `old()`

`old` 函数 [检索](https://laravel.com/docs/5.8/requests#retrieving-input) 闪存到会话的 [旧输入值](https://laravel.com/docs/5.8/requests#old-input)：

```php
$value = old('value');

$value = old('value', 'default');
```

### `optional()`

`optional` 函数接受任何参数，并允许你访问该对象上的属性或调用方法。如果给定对象为 `null`，属性和方法将返回 `null`，而不是导致错误：

```php
return optional($user->address)->street;

{!! old('name', optional($user)->name) !!}
```

`optional` 函数也接受闭包作为它的第二个参数。如果作为第一个参数提供的值不为空，则将调用闭包：

```php
return optional(User::find($id), function ($user) {
    return new DummyUser;
});
```

### `policy()`

`policy` 函数检索给定类的 [策略](https://laravel.com/docs/5.8/authorization#creating-policies) 实例：

```php
$policy = policy(App\User::class);
```

### `redirect()`

`redirect` 函数返回一个 [重定向 HTTP 响应](https://laravel.com/docs/5.8/responses#redirects)，或者在没有参数的情况下调用时返回重定向器实例：

```php
return redirect($to = null, $status = 302, $headers = [], $secure = null);

return redirect('/home');

return redirect()->route('route.name');
```

### `report()`

`report` 函数将使用一个 [异常处理器](https://laravel.com/docs/5.8/errors#the-exception-handler) 的 `report` 方法报告异常：

```php
report($e);
```

### `request()`

`request` 函数返回当前 [请求](https://laravel.com/docs/5.8/requests) 实例或获取一个输入条目：

```php
$request = request();

$value = request('key', $default);
```

### `rescue()`

`rescue` 函数执行给定的闭包并捕获在执行过程中发生的任何异常。所有捕获到的异常将发送到 [异常处理器](https://laravel.com/docs/5.8/errors#the-exception-handler) 的 `report` 方法；但是，请求将继续处理：

```php
return rescue(function () {
    return $this->method();
});
```

你还可以向 `rescue` 函数传递第二个参数。这个参数将是『默认』值，如果在执行闭包时发生异常，应该返回该值：

```php
return rescue(function () {
    return $this->method();
}, false);

return rescue(function () {
    return $this->method();
}, function () {
    return $this->failure();
});
```

### `resolve()`

`resolve` 函数使用 [服务容器](https://laravel.com/docs/5.8/container) 将给定的类或接口名称解析为其实例：

```php
$api = resolve('HelpSpot\API');
```

### `response()`

`response` 函数创建 [响应](https://laravel.com/docs/5.8/responses) 实例或获取响应工厂的实例：

```php
return response('Hello World', 200, $headers);

return response()->json(['foo' => 'bar'], 200, $headers);
```

### `retry()`

`restry` 函数尝试执行给定的回调，直到满足给定的最大尝试阈值。如果回调函数没有抛出异常，它的返回值将被返回。如果回调引发异常，则会自动重试。如果超过最大尝试计数，将抛出异常：

```php
return retry(5, function () {
    // 尝试 5 次，在两次尝试之间休息 100 毫秒...
}, 100);
```

### `session()`

`session` 函数可用于获取或设置 [会话](https://laravel.com/docs/5.8/session) 值：

```php
$value = session('key');
```

你可以通过将键 / 值对数组传递给函数来设置值：

```php
session(['chairs' => 7, 'instruments' => 3]);
```

如果没有值传递给函数，则会返回会话存储：

```php
$value = session()->get('key');

session()->put('key', $value);
```

### `tap()`

`tap` 函数接受两个参数：一个任意的 `$value` 和一个闭包。`$value` 将传递给闭包，然后由 `tap` 函数返回。闭包的返回值无关紧要：

```php
$user = tap(User::first(), function ($user) {
    $user->name = 'taylor';

    $user->save();
});
```

如果没有传递闭包给 `tap` 函数，你可以调用给定 `$value` 上的任何方法。你调用的方法的返回值总是 `$value`，而不管方法在其定义中实际返回什么。例如，Eloquent 的 `update` 方法通常返回一个整数。但是，我们可以通过 `tap` 函数链接 `update` 方法调用来强制方法返回模型本身：

```php
$user = tap($user)->update([
    'name' => $name,
    'email' => $email,
]);
```

要向类添加 `tap` 方法，你可以在类中添加 `Illuminate\Support\Traits\Tappable` 特性。此特性的 `tap` 方法接受闭包作为其惟一参数。对象实例本身将传递给闭包，然后通过 `tap` 方法返回：

```php
return $user->tap(function ($user) {
    //
});
```

### throw_if()

如果给定布尔表达式的计算结果为 `true`，`throw_if` 函数抛出给定的异常：

```php
throw_if(! Auth::user()->isAdmin(), AuthorizationException::class);

throw_if(
    ! Auth::user()->isAdmin(),
    AuthorizationException::class,
    'You are not allowed to access this page'
);
```

### `throw_unless()`

如果给定布尔表达式的计算结果为 `false`，`throw_unless` 函数抛出给定的异常：

```php
throw_unless(Auth::user()->isAdmin(), AuthorizationException::class);

throw_unless(
    Auth::user()->isAdmin(),
    AuthorizationException::class,
    'You are not allowed to access this page'
);
```

### `today()`

`today` 函数为当前日期创建一个新的 `Illuminate\Support\Carbon` 实例：

```php
$today = today();
```

### `trait_uses_recursive()`

`trait_uses_recursive` 函数返回一个 Trait 使用的所有 Trait：

```php
$traits = trait_uses_recursive(\Illuminate\Notifications\Notifiable::class);
```

### `transform()`

`transform` 在给定值上执行一个 `Closure`，如果值不是 [空的](https://laravel.com/docs/5.8/helpers#method-blank) 并返回 `Closure` 的结果：

```php
$callback = function ($value) {
    return $value * 2;
};

$result = transform(5, $callback);

// 10
```

默认值或 `Closure` 也可以作为第三个参数传递给方法。如果给定值为空，则返回此值：

```php
$result = transform(null, $callback, 'The value is blank');

// The value is blank
```

### `validator()`

`validator` 函数使用给定的参数创建新的 [验证器](https://laravel.com/docs/5.8/validation) 实例。为方便起见，你可以使用它而不是 `Validator` 外观：

```php
$validator = validator($data, $rules, $messages);
```

### `value()`

`value` 函数返回给定的值。但是，如果将 `Closure` 传递给函数，则将执行 `Closure`，然后返回其结果：

```php
$result = value(true);

// true

$result = value(function () {
    return false;
});

// false
```

### `view()`

`view` 函数将返回一个 [视图](https://laravel.com/docs/5.8/views) 实例：

```php
return view('auth.login');
```

### `with()`

`with` 函数返回给定的值。如果将 `Closure` 作为函数的第二个参数传递，则将执行 `Closure` 并返回其结果：

```php
$callback = function ($value) {
    return (is_numeric($value)) ? $value * 2 : 0;
};

$result = with(5, $callback);

// 10

$result = with(null, $callback);

// 0

$result = with(5, null);

// 5
```
