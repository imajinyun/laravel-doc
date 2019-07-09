# 队列

## 简介

{% hint style="info" %}

Laravel 现在为你的 Redis 驱动的队列提供了 Horizon，它是一个漂亮的仪表板和配置系统。查看完整的 [Horizon文档](https://laravel.com/docs/5.8/horizon) 以获得更多信息。

{% endhint %}

Laravel 队列在各种不同的队列后端提供统一的 API，例如：Beanstalk、Amazon SQS、Redis，甚至是关系数据库。队列允许你延迟处理耗时的任务，例如：直到稍后才发送电子邮件。推迟这些耗时的任务可以大幅度加快对你的应用程序的 Web 请求。

队列配置文件存储在 `config/queue.php` 中。在此文件中，你将找到框架中包含的每个队列驱动程序的连接配置，其中包括数据库、[Beanstalkd](https://kr.github.io/beanstalkd/)、[Amazon SQS](https://aws.amazon.com/sqs/)、[Redis](https://redis.io/) 以及将立即执行作业（供本地使用）的同步驱动程序。还包括 `null` 队列驱动程序，它会丢弃排队的作业。

### 连接与队列

在开始使用 Laravel 队列之前，了解『连接』和『队列』之间的区别非常重要的。在 `config/queue.php` 配置文件中，有一个 `connections` 配置选项。此选项定义与后端服务（如：Amazon SQS、Beanstalk 或 Redis）的特定连接。但是，任何给定的队列连接都可能有多个『队列』，可能被认为是不同的堆栈或成堆的排队作业。

注意，`queue` 配置文件中的每个连接配置示例都包含一个 `queue` 属性。这是将作业发送到给定连接时将被调度到默认队列。换句话说，如果在没有显式定义作业应该被调度到哪个队列的情况下调度作业，则该作业将被放置在连接配置的 `queue` 属性中定义的队列上：

```php
// 此作业将发送到『default』队列...
Job::dispatch();

// 此作业将发送到『emails』队列...
Job::dispatch()->onQueue('emails');
```

有些应用程序可能从不需要将作业推到多个队列上，而是宁愿有一个简单的队列。然而，将作业推到多个队列对于希望对作业的处理方式进行优先级排序或分段的应用程序尤其有用，因为 Laravel 队列工作者允许你指定它应该按优先级处理哪些队列。例如，如果你将作业推到一个 `high` 队列中，你可能会运行一个赋予它们更高处理优先级的工作者：

```bash
php artisan queue:work --queue=high,default
```

### 驱动事项 & 先决条件

#### Database

为了使用 `database` 队列驱动程序，你将需要一个数据库表来保存作业。要生成创建此表的迁移，运行 `queue:table` Artisan 命令。一旦迁移被创建，可以使用 `migrate` 命令迁移你的数据库：

```bash
php artisan queue:table

php artisan migrate
```

#### Redis

要使用 `redis` 队列驱动程序，你应在 `config/database.php` 配置文件中配置 Redis 数据库连接。

**Redis Cluster**

如果你的 Redis 队列连接使用一个 Redis 集群，那么你的队列名称必须包含一个 [键散列标记](https://redis.io/topics/cluster-spec#keys-hash-tags)。为了确保给定队列的所有 Redis 键都放在相同的哈希槽中，需要这样做：

```php
'redis' => [
    'driver' => 'redis',
    'connection' => 'default',
    'queue' => '{default}',
    'retry_after' => 90,
],
```

**阻塞**

当使用 Redis 队列时，你可以使用 `block_for` 配置选项来指定驱动程序在遍历工作者循环并重新轮询 Redis 数据库之前应该等待作业可用的时间。

根据你的队列负载调整此值比连续轮询 Redis 数据库以查找新作业更有效。例如，你可以将值设置为 `5` 以指示驱动程序在等待作业可用时应该阻塞 5 秒：

```php
'redis' => [
    'driver' => 'redis',
    'connection' => 'default',
    'queue' => 'default',
    'retry_after' => 90,
    'block_for' => 5,
],
```

#### 其他驱动先决条件

列出的队列驱动程序需要以下依赖项：

* Amazon SQS：`aws/aws-sdk-php ~3.0`
* Beanstalked：`pda/pheanstalk ~4.0`
* Redis：`predis/predis ~1.0`

## 创建作业

### 生成作业类

默认情况下，你的应用程序的所有可排队作业都存储在 `app/Jobs` 目录中。如果 `app/Jobs` 目录不存在，则在运行 `make:job` Artisan 命令时将创建该目录。你可以使用 Artisan CLI 生成新的排队作业：

```bash
php artisan make:job ProcessPodcast
```

生成的类将实现 `Illuminate\Contracts\Queue\ShouldQueue` 接口，向 Laravel 指示应该将作业推入队列以异步运行。

### 类结构

作业类非常简单，通常只包含由队列处理作业时调用的 `handle` 方法。首先，让我们看一个示例作业类。在本例中，我们假设管理了一个播客发布服务，需要在发布之前处理已上传的播客文件：

```php
<?php

namespace App\Jobs;

use App\Podcast;
use App\AudioProcessor;
use Illuminate\Bus\Queueable;
use Illuminate\Queue\SerializesModels;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;

class ProcessPodcast implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    protected $podcast;

    /**
     * 创建一个新的作业实例。
     *
     * @param  Podcast  $podcast
     * @return void
     */
    public function __construct(Podcast $podcast)
    {
        $this->podcast = $podcast;
    }

    /**
     * 执行作业。
     *
     * @param  AudioProcessor  $processor
     * @return void
     */
    public function handle(AudioProcessor $processor)
    {
        // 处理上传播客...
    }
}
```

在本例中，注意我们能够将一个 [Eloquent 模型](https://laravel.com/docs/5.8/eloquent) 直接传递到队列作业的构造方法中。由于作业使用 `SerializesModels` 特性，当作业处理时，Eloquent 模型将被优雅地序列化和非序列化。如果你排队的作业在其构造方法中接受一个 Eloquent 模型，那么只有模型的标识符才会被序列化到队列中。当实际处理作业时，队列系统将自动从数据库中重新检索完整的模型实例。它对你的应用程序是完全透明的，并且可以防止在序列化完整的 Eloquent 模型实例时可能出现的问题。

当作业通过队列处理时，将调用 `handle` 方法。注意，我们能够在作业的 `handle` 方法上键入提示依赖项。Laravel [服务容器](https://laravel.com/docs/5.8/container) 自动注入这些依赖项。

如果希望完全控制容器如何将依赖项注入 `handle` 方法，可以使用容器的 `bindMethod` 方法。`bindMethod` 方法接受一个回调函数，该回调函数接收作业和容器。在回调函数中，你可以随意调用 `handle` 方法。通常，你应该从一个 [服务提供者](https://laravel.com/docs/5.8/providers) 调用此方法：

```php
use App\Jobs\ProcessPodcast;

$this->app->bindMethod(ProcessPodcast::class.'@handle', function ($job, $app) {
    return $job->handle($app->make(AudioProcessor::class));
});
```

{% hint style="danger" %}

二进制数据（如原始图像内容）应该在传递到队列作业之前通过 `base64_encode` 函数传递。否则，作业放在队列中时可能无法正确序列化为 JSON。

{% endhint %}

## 调度作业

一旦编写了你的作业类，你就可以使用作业本身的 `dispatch` 方法来调度它。传递给 `dispatch` 方法的参数将被传递给作业的构造方法：

```php
<?php

namespace App\Http\Controllers;

use App\Jobs\ProcessPodcast;
use Illuminate\Http\Request;
use App\Http\Controllers\Controller;

class PodcastController extends Controller
{
    /**
     * 存储一个新的播客。
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        // 创建播客...

        ProcessPodcast::dispatch($podcast);
    }
}
```

### 延迟调度

如果希望延迟队列作业的执行，你可以在调度作业时使用 `delay` 方法。例如，让我们指定一个作业应该在发出 10 分钟之后它才被调度处理：

```php
<?php

namespace App\Http\Controllers;

use App\Jobs\ProcessPodcast;
use Illuminate\Http\Request;
use App\Http\Controllers\Controller;

class PodcastController extends Controller
{
    /**
     * 存储一个新的播客。
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        // 创建播客...

        ProcessPodcast::dispatch($podcast)
                ->delay(now()->addMinutes(10));
    }
}
```

{% hint style="danger" %}

Amazon SQS 队列服务的最大延迟时间为 15 分钟。

{% endhint %}

### 同步调度

## 排队闭包

## 运行队列工作者

## Supervisor 配置

## 处理失败的作业

## 作业事件
