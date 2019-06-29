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

## 字符串

## URL

## 杂项
