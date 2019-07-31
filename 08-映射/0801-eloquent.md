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

## 检索单个模型 / 聚合

## 插入 & 更新模型

## 删除模型

## 查询范围

## 比较模型

## 事件
