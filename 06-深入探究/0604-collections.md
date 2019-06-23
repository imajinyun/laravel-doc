# 集合

## 简介

`Illuminate\Support\Collection` 类为处理数据数组提供了一个流畅、方便的包装器。例如，检查以下代码。我们将使用 `collect` 助手从数组中创建一个新的集合实例，在每个元素上运行 `strtoupper` 函数，然后删除所有空元素：

```php
$collection = collect(['taylor', 'abigail', null])->map(function ($name) {
    return strtoupper($name);
})
->reject(function ($name) {
    return empty($name);
});
```

正如你所看到的，`Collection` 类允许你链接它的方法来执行底层数组的流畅映射和缩减。通常，集合是不可变的，这意味着每个 `Collection` 方法都返回一个全新的 `Collection` 实例。

### 创建集合

如上所述，`collect` 助手为给定数组返回一个新的 `Illuminate\Support\Collection` 实例。因此，创建集合非常简单：

```php
$collection = collect([1, 2, 3]);
```

{% hint style="info" %}

[Eloquent](https://laravel.com/docs/5.8/eloquent) 查询结果集总是作为 `Collection` 实例返回。

{% endhint %}

### 扩展集合

集合是『可宏的』，这允许你在运行时向 `Collection` 类添加其他方法。例如，下面的代码向集合类添加了一个 `toUpper` 方法：

```php
use Illuminate\Support\Str;

Collection::macro('toUpper', function () {
    return $this->map(function ($value) {
        return Str::upper($value);
    });
});

$collection = collect(['first', 'second']);

$upper = $collection->toUpper();

// ['FIRST', 'SECOND']
```

通常，你应该在 [服务提供者](https://laravel.com/docs/5.8/providers) 中声明集合宏。

## 可用的方法

对于本文档的其余部分，我们将讨论 `Collection` 类上可用的每个方法。记住，所有这些方法都可以链接起来流畅地操作底层数组。此外，几乎每个方法都返回一个新的 `Collection` 实例，允许你在必要时保留集合的原始副本：

|                                                                                  |                                                                            |                                                                              |
| -------------------------------------------------------------------------------- | -------------------------------------------------------------------------- | ---------------------------------------------------------------------------- |
| [all](https://laravel.com/docs/5.8/collections#method-all)                       | [isNotEmpty](https://laravel.com/docs/5.8/collections#method-isnotempty)   | [sort](https://laravel.com/docs/5.8/collections#method-sort)                 |
| [average](https://laravel.com/docs/5.8/collections#method-average)               | [join](https://laravel.com/docs/5.8/collections#method-join)               | [sortBy](https://laravel.com/docs/5.8/collections#method-sortby)             |
| [avg](https://laravel.com/docs/5.8/collections#method-avg)                       | [keyBy](https://laravel.com/docs/5.8/collections#method-keyby)             | [sortByDesc](https://laravel.com/docs/5.8/collections#method-keyby)          |
| [chunk](https://laravel.com/docs/5.8/collections#method-chunk)                   | [keys](https://laravel.com/docs/5.8/collections#method-keys)               | [sortKeys](https://laravel.com/docs/5.8/collections#method-sortkeys)         |
| [collapse](https://laravel.com/docs/5.8/collections#method-collapse)             | [last](https://laravel.com/docs/5.8/collections#method-last)               | [sortKeysDesc](https://laravel.com/docs/5.8/collections#method-sortkeysdesc) |
| [combine](https://laravel.com/docs/5.8/collections#method-combine)               | [macro](https://laravel.com/docs/5.8/collections#method-macro)             | [splice](https://laravel.com/docs/5.8/collections#method-splice)             |
| [concat](https://laravel.com/docs/5.8/collections#method-concat)                 | [make](https://laravel.com/docs/5.8/collections#method-make)               | [split](https://laravel.com/docs/5.8/collections#method-split)               |
| [contains](https://laravel.com/docs/5.8/collections#method-contains)             | [map](https://laravel.com/docs/5.8/collections#method-map)                 | [sum](https://laravel.com/docs/5.8/collections#method-sum)                   |
| [containsStrict](https://laravel.com/docs/5.8/collections#method-containsstrict) | [mapInto](https://laravel.com/docs/5.8/collections#method-mapinto)         | [take](https://laravel.com/docs/5.8/collections#method-take)                 |
| [count](https://laravel.com/docs/5.8/collections#method-count)                   | [mapSpread](https://laravel.com/docs/5.8/collections#method-mapspread)     | [tap](https://laravel.com/docs/5.8/collections#method-tap)                   |
| [countBy](https://laravel.com/docs/5.8/collections#method-countBy)               | [mapToGroups](https://laravel.com/docs/5.8/collections#method-maptogroups) | [times](https://laravel.com/docs/5.8/collections#method-times)               |
| [crossJoin](https://laravel.com/docs/5.8/collections#method-crossjoin)           | [mapWithKeys](https://laravel.com/docs/5.8/collections#method-mapwithkeys) | [toArray](https://laravel.com/docs/5.8/collections#method-toarray)           |
| [dd](https://laravel.com/docs/5.8/collections#method-dd)                         | [max](https://laravel.com/docs/5.8/collections#method-max)                 | [toJson](https://laravel.com/docs/5.8/collections#method-tojson)             |
| [diff](https://laravel.com/docs/5.8/collections#method-diff)                     | [median](https://laravel.com/docs/5.8/collections#method-median)           | [transform](https://laravel.com/docs/5.8/collections#method-transform)       |
| [diffAssoc](https://laravel.com/docs/5.8/collections#method-diffassoc)           | [merge](https://laravel.com/docs/5.8/collections#method-merge)             | [union](https://laravel.com/docs/5.8/collections#method-union)               |
| [diffKeys](https://laravel.com/docs/5.8/collections#method-diffkeys)             | [min](https://laravel.com/docs/5.8/collections#method-min)                 | [unique](https://laravel.com/docs/5.8/collections#method-unique)             |
| [dump](https://laravel.com/docs/5.8/collections#method-dump)                     | [mode](https://laravel.com/docs/5.8/collections#method-mode)               | [uniqueStrict](https://laravel.com/docs/5.8/collections#method-uniquestrict) |

### `all()`

`all` 方法返回通过集合所表示的底层数组：

```php
collect([1, 2, 3])->all();

// [1, 2, 3]
```

### `average()`

`avg` 方法的别名。

### `avg()`

`avg` 方法返回一个给定键的 [平均值](https://en.wikipedia.org/wiki/Average)：

```php
$average = collect([['foo' => 10], ['foo' => 10], ['foo' => 20], ['foo' => 40]])->avg('foo');

// 20

$average = collect([1, 1, 2, 4])->avg();

// 2
```

### `chunk()`

`chunk` 方法将集合分解为多个给定大小的更小的集合：

```php
$collection = collect([1, 2, 3, 4, 5, 6, 7]);

$chunks = $collection->chunk(4);

$chunks->toArray();

// [[1, 2, 3, 4], [5, 6, 7]]
```

当在工作中使用诸如 [Bootstrap](https://getbootstrap.com/docs/4.1/layout/grid/) 之类的网格系统时，这种方法在 [视图](https://laravel.com/docs/5.8/views) 中特别有用。假设你有一个 [Eloquent](https://laravel.com/docs/5.8/eloquent) 模型的集合，你想在一个网格中显示：

```php
@foreach ($products->chunk(3) as $chunk)
    <div class="row">
        @foreach ($chunk as $product)
            <div class="col-xs-4">{{ $product->name }}</div>
        @endforeach
    </div>
@endforeach
```

### `collapse()`

`collapse` 方法将数组集合折叠为单个平面集合：

```php
$collection = collect([[1, 2, 3], [4, 5, 6], [7, 8, 9]]);

$collapsed = $collection->collapse();

$collapsed->all();

// [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

### `combine()`

`combine` 方法将集合的值（作为键）与另一个数组或集合的值合并在一起：

```php
$collection = collect(['name', 'age']);

$combined = $collection->combine(['George', 29]);

$combined->all();

// ['name' => 'George', 'age' => 29]
```

### `concat()`

`concat` 方法将给定的 `array` 或集合值附加到集合的末尾：

```php
$collection = collect(['John Doe']);

$concatenated = $collection->concat(['Jane Doe'])->concat(['name' => 'Johnny Doe']);

$concatenated->all();

// ['John Doe', 'Jane Doe', 'Johnny Doe']
```

### `contains()`

`contains` 方法确定集合是否包含一个给定的项：

```php
$collection = collect(['name' => 'Desk', 'price' => 100]);

$collection->contains('Desk');

// true

$collection->contains('New York');

// false
```

你还可以将键 / 值对传递给 `contains` 方法，该方法将确定集合中是否存在给定的对：

```php
$collection = collect([
    ['product' => 'Desk', 'price' => 200],
    ['product' => 'Chair', 'price' => 100],
]);

$collection->contains('product', 'Bookcase');

// false
```

最后，你还可以将回调传递给 `contains` 方法来执行你自己的真正测试：

```php
$collection = collect([1, 2, 3, 4, 5]);

$collection->contains(function ($value, $key) {
    return $value > 5;
});

// false
```

`contains` 方法在检查条目值时使用『松散』比较，这意味着具有整数值的字符串将被认为等于具有相同值的整数。使用 [containsStrict](https://laravel.com/docs/5.8/collections#method-containsstrict) 方法使用『严格』比较进行过滤。

### `containsStrict()`

该方法具有与 [contains](https://laravel.com/docs/5.8/collections#method-contains) 方法具有相同的签名；但是，所有值都使用『严格』比较进行比较。

### `count()`

`count` 方法返回集合中条目的总数量：

```php
$collection = collect([1, 2, 3, 4]);

$collection->count();

// 4
```

### `countBy()`

`countBy` 方法计算集合中值的出现次数。默认情况下，该方法计算每个元素的出现次数：

```php
$collection = collect([1, 2, 2, 2, 3]);

$counted = $collection->countBy();

$counted->all();

// [1 => 1, 2 => 3, 3 => 1]
```

但是，你将回调传递给 `countBy` 方法，通过一个自定义值计算所有条目：

```php
$collection = collect(['alice@gmail.com', 'bob@yahoo.com', 'carlos@gmail.com']);

$counted = $collection->countBy(function ($email) {
    return substr(strrchr($email, "@"), 1);
});

$counted->all();

// ['gmail.com' => 2, 'yahoo.com' => 1]
```

### `crossJoin()`

`crossJoin` 方法在给定的数组或集合之间交叉连接集合的值，返回一个具有所有可能排列的笛卡尔积：

```php
$collection = collect([1, 2]);

$matrix = $collection->crossJoin(['a', 'b']);

$matrix->all();

/*
    [
        [1, 'a'],
        [1, 'b'],
        [2, 'a'],
        [2, 'b'],
    ]
*/

$collection = collect([1, 2]);

$matrix = $collection->crossJoin(['a', 'b'], ['I', 'II']);

$matrix->all();

/*
    [
        [1, 'a', 'I'],
        [1, 'a', 'II'],
        [1, 'b', 'I'],
        [1, 'b', 'II'],
        [2, 'a', 'I'],
        [2, 'a', 'II'],
        [2, 'b', 'I'],
        [2, 'b', 'II'],
    ]
*/
```

### `dd()`

`dd` 方法转储集合的条目并结束脚本的执行：

```php
$collection = collect(['John Doe', 'Jane Doe']);

$collection->dd();

/*
    Collection {
        #items: array:2 [
            0 => "John Doe"
            1 => "Jane Doe"
        ]
    }
*/
```

如果不想停止执行脚本，可以使用 [dump](https://laravel.com/docs/5.8/collections#method-dump) 方法替代。

### `diff()`

`diff` 方法将集合与另一集合或基于其值的普通 PHP `array` 进行比较。此方法将返回原始集合与给定集合差集的值。

```php
$collection = collect([1, 2, 3, 4, 5]);

$diff = $collection->diff([2, 4, 6, 8]);

$diff->all();

// [1, 3, 5]
```

### `diffAssoc()`

`diffAssoc` 方法将集合与另一个集合或基于其键值的普通 PHP `array` 进行比较。此方法将返回原始集合不存在于给定集合中的键 / 值（相同的键值不同）对：

```php
$collection = collect([
    'color' => 'orange',
    'type' => 'fruit',
    'remain' => 6
]);

$diff = $collection->diffAssoc([
    'color' => 'yellow',
    'type' => 'fruit',
    'remain' => 3,
    'used' => 6
]);

$diff->all();

// ['color' => 'orange', 'remain' => 6]
```

### `diffKeys()`

`diffKeys` 方法将集合与另一个集合或基于其键的普通 PHP `array` 进行比较。此方法将返回原始集合中不存在于给定集合中的键 / 值（原始集合与给定集合条目数量不相等时，将原始集合中多余的键值对填充到比较后的结果集中）对：

```php
$collection = collect([
    'one' => 10,
    'two' => 20,
    'three' => 30,
    'four' => 40,
    'five' => 50,
]);

$diff = $collection->diffKeys([
    'two' => 2,
    'four' => 4,
    'six' => 6,
    'eight' => 8,
]);

$diff->all();

// ['one' => 10, 'three' => 30, 'five' => 50]
```

### `dump()`

`dump` 方法转储集合的条目：

```php
$collection = collect(['John Doe', 'Jane Doe']);

$collection->dump();

/*
    Collection {
        #items: array:2 [
            0 => "John Doe"
            1 => "Jane Doe"
        ]
    }
*/
```

如果你想在转储集合之后停止执行脚本，请使用 [dd](https://laravel.com/docs/5.8/collections#method-dd) 方法。

### `duplicates()`

`duplicates` 方法从集合中检索并返回重复的值：

```php
$collection = collect(['a', 'b', 'a', 'c', 'b']);

$collection->duplicates();

// [ 2 => 'a', 4 => 'b' ]
```

### `each()`

`each` 方法遍历集合中的条目，并将每个条目传递给回调函数：

```php
$collection->each(function ($item, $key) {
    //
});
```

如果你希望停止遍历条目，可以从回调函数返回 `false`：

```php
$collection->each(function ($item, $key) {
    if (/* some condition */) {
        return false;
    }
});
```

### `eachSpread()`

`eachSpread` 方法遍历集合的条目，将每个嵌套的条目值传递到给定的回调函数：

```php
$collection = collect([['John Doe', 35], ['Jane Doe', 33]]);

$collection->eachSpread(function ($name, $age) {
    //
});
```

你可以通过从回调函数返回 `false` 停止遍历条目：

```php
$collection->eachSpread(function ($name, $age) {
    return false;
});
```

### `every()`

`every` 方法可用于验证集合的所有元素是否通过给定的真值测试：

```php
collect([1, 2, 3, 4])->every(function ($value, $key) {
    return $value > 2;
});

// false
```

如果集合为空，`every` 将返回 `true`：

```php
$collection = collect([]);

$collection->every(function($value, $key) {
    return $value > 2;
});

// true
```

### `except()`

`except` 方法返回集合中的所有条目，那些指定键的条目除外：

```php
$collection = collect(['product_id' => 1, 'price' => 100, 'discount' => false]);

$filtered = $collection->except(['price', 'discount']);

$filtered->all();

// ['product_id' => 1]
```

有关 `except` 的逆函数，参看 [only](https://laravel.com/docs/5.8/collections#method-only) 方法。

### `filter()`

`filter` 方法使用给定的回调过滤集合，只保留那些通过给定真值测试的条目：

```php
$collection = collect([1, 2, 3, 4]);

$filtered = $collection->filter(function ($value, $key) {
    return $value > 2;
});

$filtered->all();

// [3, 4]
```

如果没有提供回调函数，则将删除集合中与 `false` 等价的所有条目：

```php
$collection = collect([1, 2, 3, null, false, '', 0, []]);

$collection->filter()->all();

// [1, 2, 3]
```

有关 `filter` 的逆函数，查看 [reject](https://laravel.com/docs/5.8/collections#method-reject) 函数。

### `first()`

`first` 方法返回集合中传递的一个给定真值测试的第一个元素：

```php
collect([1, 2, 3, 4])->first(function ($value, $key) {
    return $value > 2;
});

// 3
```

你还可以调用 `first` 方法而没有参数来获取集合中的第一个元素。如果集合为空，则返回 `null`：

```php
collect([1, 2, 3, 4])->first();

// 1
```

### `firstWhere()`

`firstWhere` 方法返回具有给定键 / 值对的集合中的第一个元素：

```php
$collection = collect([
    ['name' => 'Regena', 'age' => null],
    ['name' => 'Linda', 'age' => 14],
    ['name' => 'Diego', 'age' => 23],
    ['name' => 'Linda', 'age' => 84],
]);

$collection->firstWhere('name', 'Linda');

// ['name' => 'Linda', 'age' => 14]
```

你也可以用一个操作符来调用 `firstWhere` 方法：

```php
$collection->firstWhere('age', '>=', 18);

// ['name' => 'Diego', 'age' => 23]
```

与 [where](https://laravel.com/docs/5.8/collections#method-where) 方法类似，你可以将一个参数传递给 `firstWhere` 方法。在这个场景中，`firstWhere` 方法将返回给定项键的值为『精确的』第一个条目：

```php
$collection->firstWhere('age');

// ['name' => 'Linda', 'age' => 14]
```

### `flatMap()`

`flatMap` 方法遍历集合并将每个值传递给给定的回调函数。回调函数可以自由地修改条目并返回它，从而形成一个修改条目的新集合。然后，将数组平铺成一层：

```php
$collection = collect([
    ['name' => 'Sally'],
    ['school' => 'Arkansas'],
    ['age' => 28]
]);

$flattened = $collection->flatMap(function ($values) {
    return array_map('strtoupper', $values);
});

$flattened->all();

// ['name' => 'SALLY', 'school' => 'ARKANSAS', 'age' => '28'];
```

### `flatten()`

`flatten` 方法将多维集合压扁为一维集合：

```php
$collection = collect(['name' => 'taylor', 'languages' => ['php', 'javascript']]);

$flattened = $collection->flatten();

$flattened->all();

// ['taylor', 'php', 'javascript'];
```

你可以选择向方法传递一个『深度』参数：

```php
$collection = collect([
    'Apple' => [
        ['name' => 'iPhone 6S', 'brand' => 'Apple'],
    ],
    'Samsung' => [
        ['name' => 'Galaxy S7', 'brand' => 'Samsung']
    ],
]);

$products = $collection->flatten(1);

$products->values()->all();

/*
    [
        ['name' => 'iPhone 6S', 'brand' => 'Apple'],
        ['name' => 'Galaxy S7', 'brand' => 'Samsung'],
    ]
*/
```

在这个例子中，调用 `flatten` 而不提供深度也会使嵌套数组变平，结果为 `['iPhone 6S'、'Apple'、'Galaxy S7'、'Samsung']`。提供一个深度允许你限制将被压扁的嵌套数组的层级。

### `flip()`

`flip` 方法将集合的键与其对应的值交换：

```php
$collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

$flipped = $collection->flip();

$flipped->all();

// ['taylor' => 'name', 'laravel' => 'framework']
```

### `forget()`

`forget` 方法根据条目的键从集合中删除条目：

```php
$collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

$collection->forget('name');

$collection->all();

// ['framework' => 'laravel']
```

{% hint style="danger" %}

与大多数其他集合方法不同，`forget` 不返回一个新的修改后的集合；它修改被调用的集合。

{% endhint %}

### `forPage()`

`forPage` 方法返回一个新集合，其中包含将出现在给定页码上的条目。该方法接受页码作为它的第一个参数，每个页面显示的条目数作为它的第二个参数：

```php
$collection = collect([1, 2, 3, 4, 5, 6, 7, 8, 9]);

$chunk = $collection->forPage(2, 3);

$chunk->all();

// [4, 5, 6]
```

### `get()`

`get` 方法返回给定键上的条目。如果键不存在，则返回 `null`：

```php
$collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

$value = $collection->get('name');

// taylor
```

你可以选择传递一个默认值作为其第二个参数：

```php
$collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

$value = $collection->get('foo', 'default-value');

// default-value
```

你甚至可以将回调作为默认值传递。如果指定的键不存在，将返回回调的结果：

```php
$collection->get('email', function () {
    return 'default-value';
});

// default-value
```

### `groupBy()`

`groupBy` 方法根据给定的键对集合的条目进行分组：

```php
$collection = collect([
    ['account_id' => 'account-x10', 'product' => 'Chair'],
    ['account_id' => 'account-x10', 'product' => 'Bookcase'],
    ['account_id' => 'account-x11', 'product' => 'Desk'],
]);

$grouped = $collection->groupBy('account_id');

$grouped->toArray();

/*
    [
        'account-x10' => [
            ['account_id' => 'account-x10', 'product' => 'Chair'],
            ['account_id' => 'account-x10', 'product' => 'Bookcase'],
        ],
        'account-x11' => [
            ['account_id' => 'account-x11', 'product' => 'Desk'],
        ],
    ]
*/
```

你可以传递回调，而不是传递字符串 `key`。回调方法应该返回你希望键入组的值：

```php
$grouped = $collection->groupBy(function ($item, $key) {
    return substr($item['account_id'], -3);
});

$grouped->toArray();

/*
    [
        'x10' => [
            ['account_id' => 'account-x10', 'product' => 'Chair'],
            ['account_id' => 'account-x10', 'product' => 'Bookcase'],
        ],
        'x11' => [
            ['account_id' => 'account-x11', 'product' => 'Desk'],
        ],
    ]
*/
```

多个分组条件可以作为数组传递。每个数组元素将应用于多维数组中的相应级别：

```php
$data = new Collection([
    10 => ['user' => 1, 'skill' => 1, 'roles' => ['Role_1', 'Role_3']],
    20 => ['user' => 2, 'skill' => 1, 'roles' => ['Role_1', 'Role_2']],
    30 => ['user' => 3, 'skill' => 2, 'roles' => ['Role_1']],
    40 => ['user' => 4, 'skill' => 2, 'roles' => ['Role_2']],
]);

$result = $data->groupBy([
    'skill',
    function ($item) {
        return $item['roles'];
    },
], $preserveKeys = true);

/*
[
    1 => [
        'Role_1' => [
            10 => ['user' => 1, 'skill' => 1, 'roles' => ['Role_1', 'Role_3']],
            20 => ['user' => 2, 'skill' => 1, 'roles' => ['Role_1', 'Role_2']],
        ],
        'Role_2' => [
            20 => ['user' => 2, 'skill' => 1, 'roles' => ['Role_1', 'Role_2']],
        ],
        'Role_3' => [
            10 => ['user' => 1, 'skill' => 1, 'roles' => ['Role_1', 'Role_3']],
        ],
    ],
    2 => [
        'Role_1' => [
            30 => ['user' => 3, 'skill' => 2, 'roles' => ['Role_1']],
        ],
        'Role_2' => [
            40 => ['user' => 4, 'skill' => 2, 'roles' => ['Role_2']],
        ],
    ],
];
*/
```

### `has`

`has` 方法确定集合中是否存在给定的键：

```php
$collection = collect(['account_id' => 1, 'product' => 'Desk', 'amount' => 5]);

$collection->has('product');

// true

$collection->has(['product', 'amount']);

// true

$collection->has(['amount', 'price']);

// false
```

### `implode()`

`implode` 方法连接集合中的条目。它的参数取决于集合中条目的类型。如果集合包含数组或对象，则应传递你希望连接的属性的键和希望放置在值之间的『粘合』字符串：

```php
$collection = collect([
    ['account_id' => 1, 'product' => 'Desk'],
    ['account_id' => 2, 'product' => 'Chair'],
]);

$collection->implode('product', ', ');

// Desk, Chair
```

### `intersect()`

`intersect` 方法从原始集合中删除在给定 `array` 或集合中不存在的任何值。结果集合将保留原始集合的键：

```php
$collection = collect(['Desk', 'Sofa', 'Chair']);

$intersect = $collection->intersect(['Desk', 'Chair', 'Bookcase']);

$intersect->all();

// [0 => 'Desk', 2 => 'Chair']
```

### `intersectByKeys()`

`intersectByKeys` 方法从原始集合中删除在给定 `array` 或集合中不存在的任何键：

```php
$collection = collect([
    'serial' => 'UX301', 'type' => 'screen', 'year' => 2009
]);

$intersect = $collection->intersectByKeys([
    'reference' => 'UX404', 'type' => 'tab', 'year' => 2011
]);

$intersect->all();

// ['type' => 'screen', 'year' => 2009]
```

### `isEmpty()`

如果集合为空 `isEmpty` 方法返回 `true`；否则，返回 `false`:

```php
collect([])->isEmpty();

// true
```

### `isNotEmpty()`

如果集合不为空 `isNotEmpty` 方法返回 `true`；否则，返回 `false`：

```php
collect([])->isNotEmpty();

// false
```

### `join()`

`join` 方法用一个字符串连接集合的值：

```php
collect(['a', 'b', 'c'])->join(', '); // 'a, b, c'
collect(['a', 'b', 'c'])->join(', ', ', and '); // 'a, b, and c'
collect(['a', 'b'])->join(', ', ' and '); // 'a and b'
collect(['a'])->join(', ', ' and '); // 'a'
collect([])->join(', ', ' and '); // ''
```

### `keyBy()`

`keyBy` 方法根据给定的键键入到集合中。如果多个条目具有相同的键，则只有最后一个键将出现在新集合中：

```php
$collection = collect([
    ['product_id' => 'prod-100', 'name' => 'Desk'],
    ['product_id' => 'prod-200', 'name' => 'Chair'],
]);

$keyed = $collection->keyBy('product_id');

$keyed->all();

/*
    [
        'prod-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
        'prod-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
    ]
*/
```

你还可以将回调传递到此方法。回调应该返回值通过以下方式键入到集合中：

```php
$keyed = $collection->keyBy(function ($item) {
    return strtoupper($item['product_id']);
});

$keyed->all();

/*
    [
        'PROD-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
        'PROD-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
    ]
*/
```

### `keys()`

`keys` 方法返回集合的所有键：

```php
$collection = collect([
    'prod-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
    'prod-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
]);

$keys = $collection->keys();

$keys->all();

// ['prod-100', 'prod-200']
```

### `last()`

`last` 方法返回集合中通过给定真值测试的最后一个元素：

```php
collect([1, 2, 3, 4])->last(function ($value, $key) {
    return $value < 3;
});

// 2
```

你还可以调用没有参数的 `last` 方法来获取集合中的最后一个元素。如果集合为空，则返回 `null`：

```php
collect([1, 2, 3, 4])->last();

// 4
```

### `sort()`

`sort` 方法对集合进行排序。排序后的集合保留原始数组键，因此在本例中，我们将使用 `values` 方法将键重置为连续编号的索引：

```php
$collection = collect([5, 3, 1, 2, 4]);

$sorted = $collection->sort();

$sorted->values()->all();

// [1, 2, 3, 4, 5]
```

如果你的排序需求更高级，你可以传递一个回调函数来使用你自己的算法进行 `sort`。参考关于 `uasort` 的 PHP 文档，它是集合的 `sort` 方法在底层调用的。

{% hint style="info" %}

如果需要对嵌套数组或对象集合排序，请参阅 `sortBy` 和 `sortByDesc` 方法。

{% endhint %}

## 高阶消息
