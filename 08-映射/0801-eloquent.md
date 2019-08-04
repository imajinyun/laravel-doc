# Eloquent：起步

## 简介

Laravel 中包含的 Eloquent ORM 提供了一个漂亮、简单的 ActiveRecord 实现，用于处理你的数据库。每个数据库表都有一个对应的『模型』，用于与该表交互。模型允许你查询表中的数据，并将新记录插入表中。

在开始之前，确保在 `config/database.php` 中配置数据库连接。有关配置数据库的更多信息，[查看文档](https://laravel.com/docs/5.8/database#configuration)。

## 定义模型

首先，让我们创建一个 Eloquent 模型。模型通常位于 `app` 目录中，但是你可以自由地将它们放在任何可以根据你的 `composer.json` 自动加载的位置。所有的 Eloquent 模型都继承 `Illuminate\Database\Eloquent\Model` 类。

创建模型实例的最简单方法是使用 `make:model` [Artisan 命令](https://laravel.com/docs/5.8/artisan)：

```bash
php artisan make:model Flight
```

如果希望在生成模型时生成 [数据库迁移](https://laravel.com/docs/5.8/migrations)，可以使用 `——migration` 或 `-m` 选项：

```bash
php artisan make:model Flight --migration

php artisan make:model Flight -m
```

### Eloquent 模型约束

现在，让我们来看一个 `Flight` 模型示例，我们将使用它从 `flights` 数据库表中检索和存储信息：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    //
}
```

#### 表名

注意，我们并没有告诉 Eloquent 应该使用哪个表作为我们的 `Flight` 模型。按照惯例，『蛇形命名』类的复数名称将用作表名，除非显式指定了另一个名称。因此，在本例中，Eloquent 将假定 `Flight` 模型将记录存储在 `flights` 表中。你可以通过在模型上定义 `table` 属性来指定自定义表：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    /**
     * 与模型关联的表。
     *
     * @var string
     */
    protected $table = 'my_flights';
}
```

#### 主键

Eloquent 还假设每个表都有一个名为 `id` 的主键列。你可以定义一个受保护的 `$primaryKey` 属性来覆盖这个约定：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    /**
     * 与表关联的主键。
     *
     * @var string
     */
    protected $primaryKey = 'flight_id';
}
```

此外，Eloquent 假定的主键是一个递增的整数值，这意味着默认情况下主键将自动转换为 `int`。如果你希望使用非递增或非数字主键，你必须在你的模型上把公开 `$incrementing` 属性设置为 `false`：

```php
<?php

class Flight extends Model
{
    /**
     * 指示 ID 是否自动递增。
     *
     * @var bool
     */
    public $incrementing = false;
}
```

如果你的主键不是整数，你应在你的模型上将受保护的 `$keyType` 属性设置为 `string`：

```php
<?php

class Flight extends Model
{
    /**
     * 自动递增 ID 的『类型』。
     *
     * @var string
     */
    protected $keyType = 'string';
}
```

#### 时间戳

默认情况下，Eloquent 会期望在你的表上存在 `created_at` 和 `updated_at` 列。如果你不希望由 Eloquent 自动管理这些列，将你的模型上的 `$timestamp` 属性设置为 `false`：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    /**
     * 指示模型是否应加时间戳。
     *
     * @var bool
     */
    public $timestamps = false;
}
```

如果你需要自定义时间戳的格式，在你的模型上设置 `$dateFormat` 属性。此属性确定日期属性如何存储在数据库中，以及当模型序列化为数组或 JSON 时它们的格式：

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

如果你需要自定义用于存储时间戳的列的名称，可以在你的模型中设置 `CREATED_AT` 和 `UPDATED_AT` 常量：

```php
<?php

class Flight extends Model
{
    const CREATED_AT = 'creation_date';
    const UPDATED_AT = 'last_update';
}
```

#### 数据库连接

默认情况下，所有的 Eloquent 模型都将使用为你的应用程序配置的默认数据库连接。如果你想为模型指定一个不同的连接，使用 `$connection` 属性：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    /**
     * 模型的连接名称。
     *
     * @var string
     */
    protected $connection = 'connection-name';
}
```

### 默认属性值

如果你想为你的模型的一些属性定义默认值，你可以在你的模型上定义一个 `$attributes` 属性：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    /**
     * 模型属性的默认值。
     *
     * @var array
     */
    protected $attributes = [
        'delayed' => false,
    ];
}
```

## 检索模型

一旦创建了模型及其 [关联它的数据库表](https://laravel.com/docs/5.8/migrations#writing-migrations)，你就可以开始从数据库中检索数据了。将每个 Eloquent 模型看作一个强大的 [查询生成器](https://laravel.com/docs/5.8/queries)，允许你流畅地查询与模型关联的数据库表。例如

```php
<?php

$flights = App\Flight::all();

foreach ($flights as $flight) {
    echo $flight->name;
}
```

**添加额外的约束**

Eloquent 的 `all` 方法将返回模型表中的所有结果。由于每个 Eloquent 模型都充当 [查询生成器](https://laravel.com/docs/5.8/queries)，你还可以向查询添加约束，然后使用 `get` 方法检索结果：

```php
$flights = App\Flight::where('active', 1)
               ->orderBy('name', 'desc')
               ->take(10)
               ->get();
```

{% hint style="info" %}

由于 Eloquent 模型是查询构建器，你应该检查 [查询构建器](https://laravel.com/docs/5.8/queries) 上可用的所有方法。你可以在 Eloquent 的查询中使用这些方法中的任何一种。

{% endhint %}

**刷新模型**

你可以使用 `fresh` 和 `refresh` 方法来刷新模型。`fresh` 方法将从数据库中重新检索模型。现有的模型实例不会受到影响：

```php
$flight = App\Flight::where('number', 'FR 900')->first();

$freshFlight = $flight->fresh();
```

`refresh` 方法将使用来自数据库的新数据重新补充现有模型。此外，所有加载的关系也将被刷新：

```php
$flight = App\Flight::where('number', 'FR 900')->first();

$flight->number = 'FR 456';

$flight->refresh();

$flight->number; // "FR 900"
```

### 集合

对于像 `all` 和 `get` 这样检索多个结果的 Eloquent 方法，将返回一个 `Illuminate\Database\Eloquent\Collection` 实例。`Collection` 类提供了 [各种有用的方法](https://laravel.com/docs/5.8/eloquent-collections#available-methods) 来处理你的 Eloquent 结果：

```php
$flights = $flights->reject(function ($flight) {
    return $flight->cancelled;
});
```

你还可以像数组一样对集合进行循环：

```php
foreach ($flights as $flight) {
    echo $flight->name;
}
```

### 分块结果

如果你需要处理数千条 Eloquent 记录，使用 `chunk` 命令。`chunk` 方法将检索一个 Eloquent 模型『块』，将它们提供给给定的 `Closure` 进行处理。在处理大型结果集时，使用 `chunk` 方法将节省内存：

```php
Flight::chunk(200, function ($flights) {
    foreach ($flights as $flight) {
        //
    }
});
```

传递给方法的第一个参数是你希望接收的每个『块』的记录数。作为第二个参数传递的闭包将为从数据库检索到的每个块调用。将执行一个数据库查询来检索传递给闭包的每个记录块。

**使用游标**

`cursor` 方法允许你使用游标遍历你的数据库记录，游标只执行一个查询。当处理大量数据时，`cursor` 方法可以极大地减少你的内存使用：

```php
foreach (Flight::where('foo', 'bar')->cursor() as $flight) {
    //
}
```

## 检索单个模型 / 聚合

除了检索给定表的所有记录外，还可以使用 `find` 或 `first` 检索单个记录。这些方法不是返回模型集合，而是返回单个模型实例：

```php
// 通过主键检索模型...
$flight = App\Flight::find(1);

// 检索匹配查询约束的第一个模型...
$flight = App\Flight::where('active', 1)->first();
```

你也可以使用主键数组调用 `find` 方法，这将返回匹配记录的集合：

```php
$flights = App\Flight::find([1, 2, 3]);
```

**未发现的异常**

有时，如果一个模型没有找到，你可能希望抛出异常。这在路由或控制器中特别有用。`findOrFail` 和 `firstOrFail` 方法将检索查询的第一个结果；但是，如果没有找到结果，将抛出一个 `Illuminate\Database\Eloquent\ModelNotFoundException` 异常：

```php
$model = App\Flight::findOrFail(1);

$model = App\Flight::where('legs', '>', 100)->firstOrFail();
```

如果没有捕获异常，`404` HTTP 响应将自动发送回用户。使用这些方法时，没有必要编写显式检查来返回 `404` 响应：

```php
Route::get('/api/flights/{id}', function ($id) {
    return App\Flight::findOrFail($id);
});
```

### 检索聚合

你还可以使用 [查询生成器](https://laravel.com/docs/5.8/queries) 提供的 `count`、`sum`、`max` 和其他 [聚合方法](https://laravel.com/docs/5.8/queries#aggregates)。这些方法返回适当的标量值，而不是完整的模型实例：

```php
$count = App\Flight::where('active', 1)->count();

$max = App\Flight::where('active', 1)->max('price');
```

## 插入 & 更新模型

### 插入

要在数据库中创建一条新记录，创建一个新模型实例，在模型上设置属性，然后调用 `save` 方法：

```php
<?php

namespace App\Http\Controllers;

use App\Flight;
use Illuminate\Http\Request;
use App\Http\Controllers\Controller;

class FlightController extends Controller
{
    /**
     * 创建一个新的 flight 实例。
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        // 验证请求...

        $flight = new Flight;

        $flight->name = $request->name;

        $flight->save();
    }
}
```

在此示例中，我们将传入 HTTP 请求中的 `name` 参数分配给 `App\Flight` 模型实例的 `name` 属性。当我们调用 `save` 方法时，会将一条记录插入到数据库中。调用 `save` 方法时，将自动设置 `created_at` 和 `updated_at` 时间戳，因此无需手动设置它们。

### 更新

`save` 方法还可以用于更新数据库中已经存在的模型。要更新模型，你应该检索它，设置希望更新的任何属性，然后调用 `save` 方法。同样，`updated_at` 时间戳将自动更新，因此不需要手动设置其值：

```php
$flight = App\Flight::find(1);

$flight->name = 'New Flight Name';

$flight->save();
```

**批量更新**

你还可以对匹配给定查询的任意数量的模型执行更新。在本例中，所有 `active` 和 `destination` 为 `San Diego` 的航班都将被标记为延迟：

```php
App\Flight::where('active', 1)
          ->where('destination', 'San Diego')
          ->update(['delayed' => 1]);
```

`update` 方法需要一个列数组和值对，表示应该更新的列。

{% hint style="danger" %}

当通过 Eloquent 发布大规模更新时，不会为更新的模型触发 `saving`、`saved`、`updating` 和 `updated` 的模型事件。这是因为在发布大规模更新时，从来不会实际检索模型。

{% endhint %}

### 批量分配

你还可以使用 `create` 方法在一行中保存一个新模型。插入的模型实例将从方法返回给你。然而，在这样做之前，你需要在模型上指定一个 `fillable` 或 `guarded` 属性，因为所有的 Eloquent 模型在缺省情况下都防止大规模分配。

当用户通过请求传递一个不期望的 HTTP 参数，而该参数更改你的数据库中你不期望的列时，就会发生大规模分配漏洞。例如，恶意用户可能通过 HTTP 请求发送 `is admin` 参数，然后将其传递到你的模型的 `create` 方法中，允许用户将自己升级为管理员。

因此，要开始使用，你应该定义要进行批量分配的模型属性。你可以使用模型上的 `$fillable` 属性来这样做。例如，让我们的 `Flight` 模型的 `name` 属性可分配：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    /**
     * 可批量分配的属性。
     *
     * @var array
     */
    protected $fillable = ['name'];
}
```

一旦我们使属性批量可分配，就可以使用 `create` 方法在数据库中插入一条新记录。`create` 方法返回保存的模型实例：

```php
$flight = App\Flight::create(['name' => 'Flight 10']);
```

如果已经有一个模型实例，你可以使用 `fill` 方法用属性数组填充它：

```php
$flight->fill(['name' => 'Flight 22']);
```

**保护属性**

虽然 `$fillable` 作为一个『白名单』的属性，应该是大规模可分配的，你也可以选择使用 `$guarded`。`$guarded` 属性应该包含你不想去批量可分配的属性数组。数组中没有的所有其他属性都是批量可分配的。因此，`$guarded` 的功能类似于『黑名单』。重要的是，你应该使用 `$fillable` 或 `$guard` — 而不是两者都使用。在下面的例子中，**除了** `price` 之外的所有属性都是批量可分配的：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    /**
     * 不能批量可分配的属性。
     *
     * @var array
     */
    protected $guarded = ['price'];
}
```

如果你希望所有属性都批量可分配，可以将 `$guarded` 属性定义为空数组：

```php
/**
 * 不能批量可分配的属性。
 *
 * @var array
 */
protected $guarded = [];
```

### 其它创建方法

#### `firstOrCreate` / `firstOrNew`

你可以使用另外两种方法通过批量分配属性来创建模型：`firstOrCreate` 和 `firstOrNew`。`firstOrCreate` 方法将尝试使用给定的列 / 值对定位数据库记录。如果在数据库中找不到该模型，则将插入一条记录，其中包含第一个参数的属性以及可选的第二个参数中的属性。

与 `firstOrCreate` 类似，`firstOrNew` 方法将尝试在数据库中定位与给定属性匹配的记录。但是，如果没有找到模型，将返回一个新的模型实例。注意，`firstOrNew` 返回的模型还没有持久化到数据库中。你将需要手动调用 `save` 来保存它：

```php
// 按名称检索航班，如果不存在则创建航班...
$flight = App\Flight::firstOrCreate(['name' => 'Flight 10']);

// 按名称检索航班，或使用名称，延迟和到达时间属性创建航班...
$flight = App\Flight::firstOrCreate(
    ['name' => 'Flight 10'],
    ['delayed' => 1, 'arrival_time' => '11:30']
);

// 按名称检索，或实例化...
$flight = App\Flight::firstOrNew(['name' => 'Flight 10']);

// 按名称检索，或使用名称、延迟和到达时间属性实例化...
$flight = App\Flight::firstOrNew(
    ['name' => 'Flight 10'],
    ['delayed' => 1, 'arrival_time' => '11:30']
);
```

#### `updateOrCreate`

你还可能遇到你想去更新现有模型或创建新模型（如果不存在）的情况。Laravel 提供了一个 `updateOrCreate` 方法，可以一步完成此任务。与 `firstOrCreate` 方法一样，`updateOrCreate` 持久化模型，因此不需要调用 `save()`：

```php
// 如果有从 Oakland 到 San Diego 的航班，把价格定在 99 美元。
// 如果没有匹配的模型，创建一个。
$flight = App\Flight::updateOrCreate(
    ['departure' => 'Oakland', 'destination' => 'San Diego'],
    ['price' => 99, 'discounted' => 1]
);
```

## 删除模型

要删除模型，在模型实例上调用 `delete` 方法：

```php
$flight = App\Flight::find(1);

$flight->delete();
```

**按键删除现有模型**

在上面的例子中，我们在调用 `delete` 方法之前从数据库中检索模型。但是，如果你知道模型的主键，你可以删除模型，而不用调用 `destroy` 方法来检索它。除了一个主键作为参数外，`destroy` 方法还将接受多个主键、一个主键数组或一个主键 [集合](https://laravel.com/docs/5.8/collections)：

```php
App\Flight::destroy(1);

App\Flight::destroy(1, 2, 3);

App\Flight::destroy([1, 2, 3]);

App\Flight::destroy(collect([1, 2, 3]));
```

**通过查询删除模型**

你还可以在一组模型上运行 `delete` 语句。在本例中，我们将删除所有标记为非活动的航班。与批量更新一样，批量删除不会为被删除的模型触发任何模型事件：

```php
$deletedRows = App\Flight::where('active', 0)->delete();
```

{% hint style="danger" %}

当通过 Eloquent 执行一个批量删除语句时，`deleting` 和 `deleted` 的模型事件不会为被删除的模型触发。这是因为在执行删除语句时从来不会实际检索模型。

{% endhint %}

### 软删除

除了从数据库中实际删除记录之外，Eloquent 还可以『软删除』模型。当模型被软删除时，它们实际上并没有从你的数据库中删除。相反，在模型上设置一个 `deleted_at` 属性并将其插入数据库。如果一个模型有一个非空的 `deleted_at` 值，则该模型已被软删除。要启用模型的软删除，使用模型上的 `Illuminate\Database\Eloquent\SoftDeletes` 特性：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class Flight extends Model
{
    use SoftDeletes;
}
```

{% hint style="info" %}

`SoftDeletes` 特性会自动将 `deleted_at` 属性强制转换为 `DateTime` / `Carbon` 实例。

{% endhint %}

你应该将 `deleted_at` 列添加到你的数据库表中。Laravel [模式构建器](https://laravel.com/docs/5.8/migrations) 包含一个助手方法来创建这个列：

```php
Schema::table('flights', function (Blueprint $table) {
    $table->softDeletes();
});
```

现在，当你调用模型上的 `delete` 方法时，`deleted_at` 列将被设置为当前日期和时间。而且，当查询使用软删除的模型时，软删除的模型将自动从所有查询结果中排除。

要确定给定的模型实例是否已被软删除，使用 `trashed` 方法：

```php
if ($flight->trashed()) {
    //
}
```

### 查询软删除的模型

#### 包括软删除模型

如上所述，软删除模型将自动从查询结果中被排除。但是，你可以使用查询上的 `withTrashed` 方法强制软删除模型出现在结果集中：

```php
$flights = App\Flight::withTrashed()
                ->where('account_id', 1)
                ->get();
```

`withTrashed` 方法也可以用于 [关系](https://laravel.com/docs/5.8/eloquent-relationships) 查询：

```php
$flight->history()->withTrashed()->get();
```

#### 只检索软删除的模型

`onlyTrashed` 方法只检索软删除的模型：

```php
$flights = App\Flight::onlyTrashed()
                ->where('airline_id', 1)
                ->get();
```

#### 恢复软删除的模型

有时你可能希望『取消删除』软删除的模型。要将软删除的模型还原为活动状态，在模型实例上使用 `restore` 方法：

```php
$flight->restore();
```

你还可以在查询中使用 `restore` 方法来快速恢复多个模型。同样，与其他『批量』操作一样，这不会为恢复的模型触发任何模型事件：

```php
App\Flight::withTrashed()
        ->where('airline_id', 1)
        ->restore();
```

与 `withTrashed` 方法一样，`restore` 方法也可以用于 [关系](https://laravel.com/docs/5.8/eloquent-relationships)：

```php
$flight->history()->restore();
```

#### 永久删除模型

有时你可能需要从你的数据库中真正删除一个模型。要从数据库中永久删除软删除的模型，使用 `forceDelete` 方法：

```php
// 强制删除单个模型实例...
$flight->forceDelete();

// 强制删除所有相关模型...
$flight->history()->forceDelete();
```

## 查询范围

### 全局范围

全局范围允许你为给定模型的所有查询添加约束。Laravel 自己的 [软删除](https://laravel.com/docs/5.8/eloquent#soft-deleting) 功能利用全局范围只从数据库中提取『未删除』模型。编写你自己的全局范围可以提供一种方便、简单的方法来确保给定模型的每个查询都接受特定的约束。

#### 编写全局范围

编写全局范围很简单。定义一个实现 `Illuminate\Database\Eloquent\Scope` 接口的类。这个接口要求你实现一个方法：`apply`。`apply` 方法可以根据需要向查询添加 `where` 约束：

```php
<?php

namespace App\Scopes;

use Illuminate\Database\Eloquent\Scope;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Builder;

class AgeScope implements Scope
{
    /**
     * 将范围应用于给定的 Eloquent 查询构建器。
     *
     * @param  \Illuminate\Database\Eloquent\Builder  $builder
     * @param  \Illuminate\Database\Eloquent\Model  $model
     * @return void
     */
    public function apply(Builder $builder, Model $model)
    {
        $builder->where('age', '>', 200);
    }
}
```

{% hint style="info" %}

如果你的全局范围是在查询的查询子句中添加列，你应使用 `addSelect` 方法而不是 `select`。这将防止无意中替换查询的现有查询子句。

{% endhint %}

#### 应用全局范围

要将全局范围分配给模型，你应该覆盖给定模型的 `boot` 方法并使用 `addGlobalScope` 方法：

```php
<?php

namespace App;

use App\Scopes\AgeScope;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * 模型的『引导』方法。
     *
     * @return void
     */
    protected static function boot()
    {
        parent::boot();

        static::addGlobalScope(new AgeScope);
    }
}
```

在添加范围之后，`User::all()` 的查询将生成以下 SQL：

```sql
select * from `users` where `age` > 200
```

#### 匿名全局范围

Eloquent 还允许你使用闭包定义全局范围，这对于不保证单独类的简单范围特别有用：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Builder;

class User extends Model
{
    /**
     * 模型的『引导』方法。
     *
     * @return void
     */
    protected static function boot()
    {
        parent::boot();

        static::addGlobalScope('age', function (Builder $builder) {
            $builder->where('age', '>', 200);
        });
    }
}
```

#### 移除全局范围

如果你希望为给定查询删除全局范围，可以使用 `withoutGlobalScope` 方法。该方法接受全局范围的类名作为其惟一参数：

```php
User::withoutGlobalScope(AgeScope::class)->get();
```

或者，如果使用闭包定义全局范围：

```php
User::withoutGlobalScope('age')->get();
```

如果你想删除几个甚至所有全局作用域，你可以使用 `withoutGlobalScope` 方法：

```php
// 移除所有的全局范围...
User::withoutGlobalScopes()->get();

// 移除一些全局范围...
User::withoutGlobalScopes([
    FirstScope::class, SecondScope::class
])->get();
```

### 局部范围

局部作用域允许你定义公共约束集，你可以在你的整个应用程序中轻松重用这些约束集。例如，你可能需要经常检索所有被认为是『热门』的所有用户。要定义范围，在 Eloquent 模型的方法前面加上范围前缀。

作用域应该总是返回一个查询生成器实例：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * Scope a query to only include popular users.
     * 范围查询仅包括热门的用户。
     *
     * @param  \Illuminate\Database\Eloquent\Builder  $query
     * @return \Illuminate\Database\Eloquent\Builder
     */
    public function scopePopular($query)
    {
        return $query->where('votes', '>', 100);
    }

    /**
     * 范围查询仅包括激活的用户。
     *
     * @param  \Illuminate\Database\Eloquent\Builder  $query
     * @return \Illuminate\Database\Eloquent\Builder
     */
    public function scopeActive($query)
    {
        return $query->where('active', 1);
    }
}
```

#### 利用局部范围

一旦定义了范围，你可以在查询模型时调用范围方法。但是，在调用方法时不应该包含 `scope` 前缀。例如，你甚至可以将调用链接到各种范围：

```php
$users = App\User::popular()->active()->orderBy('created_at')->get();
```

通过 `or` 查询操作符组合多个 Eloquent 模型范围可能需要使用闭包回调：

```php
$users = App\User::popular()->orWhere(function (Builder $query) {
    $query->active();
})->get();
```

然而，由于这可能很麻烦，Laravel 提供了一个『更高阶』的 `orWhere` 方法，允许你流畅地将这些范围链接在一起，而无需使用闭包：

```php
$users = App\User::popular()->orWhere->active()->get();
```

#### 动态范围

有时，你可能希望定义一个接受参数的范围。首先，将你的附加参数添加到你的范围中。范围参数应该定义在 `$query` 参数之后：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * 范围查询仅包括给定类型的用户。
     *
     * @param  \Illuminate\Database\Eloquent\Builder  $query
     * @param  mixed  $type
     * @return \Illuminate\Database\Eloquent\Builder
     */
    public function scopeOfType($query, $type)
    {
        return $query->where('type', $type);
    }
}
```

现在，你可以在调用范围时传递参数：

```php
$users = App\User::ofType('admin')->get();
```

## 比较模型

有时你可能需要确定两个模型是否『相同』。`is` 方法可用于快速验证两个模型具有相同的主键、表和数据库连接：

```php
if ($post->is($anotherPost)) {
    //
}
```

## 事件

Eloquent 模型可以触发多个事件，允许你在模型的生命周期中钩上以下几点：`retrieved`、`creating`、`created`、`updating`、`updated`、`saving`、`saved`、`deleting`、`deleted`、`restoring`、`restored`。事件允许你在每次在数据库中保存或更新特定模型类时轻松执行代码。每个事件通过其构造函数接收模型的实例。

从数据库中检索现有模型时，将触发 `retrieved` 事件。当首次保存新模型时，将触发 `creating` 和 `created` 事件。如果数据库中已存在模型并且调用了 `save` 方法，则将触发 `updating` / `updated` 事件。但是，在这两种情况下，`saving` / `saved` 的事件将触发。

{% hint style="dagner" %}

通过 Eloquent 发布批量更新时，不会为更新的模型触发 `saved` 和 `updated` 模型事件。这是因为在发布批量更新时，实际上从未检索过模型。

{% endhint %}

首先，在你的 Eloquent 模型上定义 `$dispatchesEvents` 属性，将 Eloquent 模型生命周期的各个点映射到你自己的 [事件类](https://laravel.com/docs/5.8/events)：

```php
<?php

namespace App;

use App\Events\UserSaved;
use App\Events\UserDeleted;
use Illuminate\Notifications\Notifiable;
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable
{
    use Notifiable;

    /**
     * 模型的事件映射。
     *
     * @var array
     */
    protected $dispatchesEvents = [
        'saved' => UserSaved::class,
        'deleted' => UserDeleted::class,
    ];
}
```

定义并映射你的 Eloquent 事件之后，可以使用 [事件监听器](https://laravel.com/docs/5.8/events#defining-listeners) 来处理事件。

### 观察者

#### 定义观察者

如果你正在监听给定模型上的许多事件，你可以使用观察者将你的所有监听器分组到一个类中。观察者类的方法名反映了你希望监听的 Eloquent 事件。这些方法中的每一个都接受模型作为它们唯一的参数。`make:observer` Artisan 命令是创建新的观察者类的最简单方法：

```bash
php artisan make:observer UserObserver --model=User
```

此命令将新观察者放在 `App/Observers` 目录中。如果此目录不存在，Artisan 将为你创建该目录。你的新观察者将如下所示：

```php
<?php

namespace App\Observers;

use App\User;

class UserObserver
{
    /**
     * 处理用户『created』事件。
     *
     * @param  \App\User  $user
     * @return void
     */
    public function created(User $user)
    {
        //
    }

    /**
     * 处理用户『updated』事件。
     *
     * @param  \App\User  $user
     * @return void
     */
    public function updated(User $user)
    {
        //
    }

    /**
     * 处理用户『deleted』事件。
     *
     * @param  \App\User  $user
     * @return void
     */
    public function deleted(User $user)
    {
        //
    }
}
```

要注册观察者，在你希望观察的模型上使用 `observe` 方法。你可以在你的某个服务提供者 `boot` 方法中注册观察者。在本例中，我们将在 `AppServiceProvider` 中注册观察者：

```php
<?php

namespace App\Providers;

use App\User;
use App\Observers\UserObserver;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * 注册任何应用程序服务。
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
        User::observe(UserObserver::class);
    }
}
```
