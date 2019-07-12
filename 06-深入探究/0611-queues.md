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

如果希望立即（同步地）调度作业，你可以使用 `dispatchNow` 方法。当使用此方法时，作业不会排队，并将立即在当前进程中处理：

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use App\Jobs\ProcessPodcast;
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

        ProcessPodcast::dispatchNow($podcast);
    }
}
```

### 作业链

作业链允许你指定应该按顺序运行的排队作业列表。如果序列中的一个作业失败，其他作业将不会运行。要执行排队作业链，你可以对任何你的可调度的作业使用 `withChain` 方法：

```php
ProcessPodcast::withChain([
    new OptimizePodcast,
    new ReleasePodcast
])->dispatch();
```

{% hint style="danger" %}

使用 `$this->delete()` 方法删除作业将不会阻止正在处理的链式作业。只有当链中的作业失败时，链才会停止执行。

{% endhint %}

#### 链连接 & 队列

如果你要指定应用于链作业的默认连接和队列，可以使用 `allOnConnection` 和 `allOnQueue` 方法。除非为排队作业显式分配了不同的连接 / 队列，否则这些方法指定应使用的队列连接和队列名称：

```php
ProcessPodcast::withChain([
    new OptimizePodcast,
    new ReleasePodcast
])->dispatch()->allOnConnection('redis')->allOnQueue('podcasts');
```

### 自定义队列 & 连接

#### 调度到特定队列

通过将作业推入不同的队列，你可以对排队的作业进行『分类』，甚至可以为分配给不同队列的作业确定优先级。记住，这并不会将作业推到你的队列配置文件中定义的不同队列『连接』，而是只推到单个连接中的特定队列。要指定队列，请在调度作业时使用 `onQueue` 方法：

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

        ProcessPodcast::dispatch($podcast)->onQueue('processing');
    }
}
```

#### 调度到特定的连接

如果正在处理多个队列连接，你可以指定要将作业推送到哪个连接。要指定连接，在调度作业时使用 `onConnection` 方法：

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

        ProcessPodcast::dispatch($podcast)->onConnection('sqs');
    }
}
```

你可以链接 `onConnection` 和 `onQueue` 方法来指定作业的连接和队列：

```php
ProcessPodcast::dispatch($podcast)
              ->onConnection('sqs')
              ->onQueue('processing');
```

或者，你可以将 `connection` 指定为作业类上的一个属性：

```php
<?php

namespace App\Jobs;

class ProcessPodcast implements ShouldQueue
{
    /**
     * 应该处理作业的队列连接。
     *
     * @var string
     */
    public $connection = 'sqs';
}
```

### 指定最大作业尝试次次数 / 超时值

#### 最大尝试次数

指定作业可能尝试的最大次数的一种方法是通过 Artisan 命令行上的 `——tries` 开关：

```bash
php artisan queue:work --tries=3
```

但是，你可以采用更细粒度的方法，定义作业类本身的最大尝试次数。如果在作业上指定了最大尝试次数，那么它将优先于命令行上提供的值：

```php
<?php

namespace App\Jobs;

class ProcessPodcast implements ShouldQueue
{
    /**
     * 作业可能尝试的次数。
     *
     * @var int
     */
    public $tries = 5;
}
```

#### 基于时间的尝试

除了定义作业失败前可能尝试的多少次之外，你还可以定义作业应该超时的时间。这允许在给定的时间范围内尝试任意次数的作业。要定义作业超时的时间，在作业类中添加 `retryUntil` 方法：

```php
/**
 * 确定作业应当超时的时间。
 *
 * @return \DateTime
 */
public function retryUntil()
{
    return now()->addSeconds(5);
}
```

{% hint style="info" %}

你还可以在队事件监听器上定义 `retryUntil` 方法。

{% endhint %}

#### 超时

{% hint style="danger" %}

`timeout` 特性针对 PHP 7.1+ 和 `pcntl` PHP 扩展进行了优化。

{% endhint %}

同样，作业可以运行的最大秒数可以使用 Artisan 命令行上的 `——timeout` 开关指定：

```php
php artisan queue:work --timeout=30
```

不过，你还可以定义作业应该允许在作业类本身上运行的最大秒数。如果在作业上指定了超时，它将优先于命令行上指定的任何超时：

```php
<?php

namespace App\Jobs;

class ProcessPodcast implements ShouldQueue
{
    /**
     * 超时前作业可以运行的秒数。
     *
     * @var int
     */
    public $timeout = 120;
}
```

### 速率限制

{% hint style="danger" %}

该特性要求你的应用程序能够与 [Redis 服务器](https://laravel.com/docs/5.8/redis) 交互。

{% endhint %}

如果你的应用程序与 Redis 交互，你可能会按时间或并发性限制排队作业。当你排队的作业与同样速率受限的 API 交互时，此特性可以提供帮助。

例如，使用 `throttle` 方法，你可以将给定类型的作业节流到每 60 秒只运行 10 次。如果无法获得锁，通常你应该将作业释放回队列，以便稍后重试：

```php
Redis::throttle('key')->allow(10)->every(60)->then(function () {
    // 作业逻辑...
}, function () {
    // 不能获得锁...

    return $this->release(10);
});
```

{% hint style="info" %}

在上面的示例中，`key` 可以是惟一标识的你要对其进行速率限制的作业类型的任何字符串。例如，你可能希望基于作业的类名和它所操作的 Eloquent 模型的 ID 来构造密钥。

{% endhint %}

{% hint style="danger" %}

将节流作业释放回队列仍然会增加该作业的总 `attempts` 次数。

{% endhint %}

或者，你可以指定可以同时处理给定作业的工作者的最大数量。当队列作业正在修改一个每次只能由一个作业修改的资源时，这是很有帮助的。例如，使用 `funnel` 方法，你可以将给定类型的作业限制为一次只能由一个工作者处理：

```php
Redis::funnel('key')->limit(1)->then(function () {
    // 作业逻辑...
}, function () {
    // 不能获取锁...

    return $this->release(10);
});
```

{% hint style="info" %}

当使用速率限制时，很难确定作业成功运行所需的尝试次数。因此，将速率限制与 [基于时间的尝试](https://laravel.com/docs/5.8/queues#time-based-attempts) 相结合是有用的。

### 错误处理

如果作业被处理时抛出异常，作业将自动释放回队列，以便于它可能会再次尝试。此作业将继续被释放，直到你的应用程序允许的最大尝试次数。尝试的最大次数由 `queue:work` Artisan 命令上使用的 `——tries` 开关定义。或者，可以在作业类本身上定义最大尝试次数。有关运行队列工作者的更多信息 [可以在下面找到](https://laravel.com/docs/5.8/queues#running-the-queue-worker)。

## 排队闭包

你还可以调度闭包，而不是将作业类调度到队列。这对于需要在当前请求周期之外执行的快速、简单的任务非常有用：

```php
$podcast = App\Podcast::find(1);

dispatch(function () use ($podcast) {
    $podcast->publish();
});
```

当调度闭包到队列时，闭包的代码内容是加密签名的，因此不能在传输过程中修改。

## 运行队列工作者

Laravel 包含一个新作业被推入队列时处理它们的队列工作者。你可以使用 `queue:work` Artisan 命令运行工作者。注意，一旦 `queue:work` 命令启动，它将继续运行，直到手动停止或关闭终端：

```php
php artisan queue:work
```

{% hint style="info" %}

要保持 `queue:work` 进程在后台永久运行，你应该使用进程监视器（如：[Supervisor](https://laravel.com/docs/5.8/queues#supervisor-configuration)）来确保队列工作者不会停止运行。

{% endhint %}

记住，队列工作者是长生命周期的进程，并将引导的应用程序状态存储在内存中。因此，队列工作者在启动之后，不会注意到你的代码库中的更改。因此，在你的部署过程中，确保 [重新启动你的队列工作者](https://laravel.com/docs/5.8/queues#queue-workers-and-deployment)。

**指定连接 & 队列**

你还可以指定工作者应使用的队列连接。传递给 `work` 命令的连接名称应对应于你的 `config/queue.php` 配置文件中定义的连接之一：

```php
php artisan queue:work redis
```

你可以通过只处理给定连接的特定队列来进一步定制你的队列工作者。例如，如果所有电子邮件都在你的 `redis` 队列连接上的 `emails` 队列中处理，则你可以发出以下命令来启动只处理该队列的工作者：

```bash
php artisan queue:work redis --queue=emails
```

**处理单个作业**

`--once` 选项可以被用来指示工作者仅处理来自队列中的单个作业：

```bash
php artisan queue:work --once
```

**处理所有排队的作业 & 然后退出**

`--stop-when-empty` 选项可用于指示工作者处理所有作业，然后优雅地退出。当在一个 Docker 容器中处理 Laravel 队列时，如果希望在队列为空后关闭容器，此选项非常有用：

```bash
php artisan queue:work --stop-when-empty
```

**资源考虑因素**

守护进程队列工作者在处理每个作业之前不会『重启』框架。因此，你应该在每个作业完成后释放任何繁重的资源。例如，如果你正在使用 GD 库进行图像操作，那么在完成之后，你应该使用 `imagedestroy` 释放内存。

### 队列优化级

有时你可能希望去优先如何处理你的队列。例如，在 `config/queue.php` 中，你可以将你的 `redis` 连接的默认 `queue` 设置为 `low`。但是，有时你可能希望将作业推送到 `high` 优先级队列，如下所示：

```php
dispatch((new Job)->onQueue('high'));
```

要启动一个工作者，该工作者在继续处理 `low` 队列上的任何作业之前，验证所有的 `high` 队列作业都被处理了，请将一个逗号分隔的队列名称列表传递给 `work` 命令：

```bash
php artisan queue:work --queue=high,low
```

### 队列工作者 & 部署

由于队列工作程序是长生命周期的进程，因此它们在不重新启动的情况下不会接受对你的代码的更改。因此，使用队列工作者部署应用程序的最简单方法是在部署过程中重启工作者。你可以通过发出 `queue:restart` 命令优雅地重新启动所有工作者：

```bash
php artisan queue:restart
```

此命令将指示所有队列工作者在处理完当前作业后优雅地『死亡』，这样就不存在作业丢失。由于队列工作者将在执行 `queue:restart` 命令时死亡，因此你应该运行一个进程管理器（如：[Supervisor](https://laravel.com/docs/5.8/queues#supervisor-configuration)）来自动重新启动队列工作者。

{% hint style="info" %}

队列使用 [缓存](https://laravel.com/docs/5.8/cache) 存储重启信号，因此在使用此特性之前，你应该验证缓存驱动程序是否已为应用程序正确配置。

{% endhint %}

### 作业过期 & 超时

#### 作业过期

在你的 `config/queue.php` 配置文件中，每个队列连接都定义了一个 `retry_after` 选项。此选项指定在重试正在处理的作业之前队列连接应等待多少秒。例如，如果 `retry_after` 的值设置为 `90`，则如果作业处理了 90 秒而未被删除，则作业将被释放回队列。通常，你应将 `retry_after` 值设置为作业合理完成处理所需的最大秒数。

{% hint style="danger" %}

惟一不包含 `retry_after` 值的队列连接是 Amazon SQS。SQS 将根据 AWS 控制台中管理的 [默认可见性超时](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/AboutVT.html) 重试作业。

{% endhint %}

#### 工作者超时

`queue:work` Artisan 命令暴露一个 `——timeout` 选项。`--timeout` 选项指定 Laravel 队列主进程将等待多长时间，然后终止正在处理作业的子队列工作者。有时，由于各种原因，子队列进程可能会『冻结』，例如：没有响应的外部 HTTP 调用。`--timeout` 选项删除已超过指定时间限制的冻结进程：

```bash
php artisan queue:work --timeout=60
```

`retry_after` 配置选项和 `——timeout` CLI 选项是不同的，但它们共同确保不会丢失作业，并且只成功处理一次作业。

{% hint style="danger" %}

`--timeout` 值应该总是比你的 `retry_after` 配置值至少短几秒。这将确保处理给定作业的工作者总是在重试作业之前被杀死。如果你的 `——timeout` 选项比你的 `retry_after` 配置值长，你的作业可能会被处理两次。

{% endhint %}

### 工作者休眠时间

当作业在队列中可用时，工作者将继续处理作业，而不会在它们之间有任何延迟。然而，`sleep` 选项决定了如果没有新作业可用，工作者将『休眠』多长时间（以秒为单位）。在休眠时，工作者不会处理任何新作业——这些作业将在工作者再次唤醒后处理。

```bash
php artisan queue:work --sleep=3
```

## Supervisor 配置

### 安装 Supervisor

Supervisor 是 Linux 操作系统的进程监视器，如果队列工作进程失败，它将自动重新启动你的 `queue:work`。要在 Ubuntu 上安装 Supervisor，你可以使用下面的命令：

```bash
sudo apt-get install supervisor
```

{% hint style="info" %}

如果你自己配置 Supervisor 听起来很困难，请考虑使用 [Laravel Forge](https://forge.laravel.com/)，它将自动为你的 Laravel 项目安装和配置 Supervisor。

{% endhint %}

### 配置 Supervisor

Supervisor 配置文件通常存储在 `/etc/supervisor/conf.d` 目录中。在此目录中，你可以创建任意数量的配置文件，指示 Supervisor 如何监视你的进程。例如，让我们创建一个 `laravel-worker.conf` 文件来启动和监视 `queue:work` 进程：

```ini
[program:laravel-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /home/forge/app.com/artisan queue:work sqs --sleep=3 --tries=3
autostart=true
autorestart=true
user=forge
numprocs=8
redirect_stderr=true
stdout_logfile=/home/forge/app.com/worker.log
```

在本例中，`numprocs` 指令将指示 Supervisor 运行 8 个 `queue:work` 进程并监视它们，如果它们失败，则自动重新启动它们。你应该更改 `command` 指令的 `queue:work sqs` 部分，以反映所需的队列连接。

### 启动 Supervisor

一旦创建了配置文件，你就可以使用以下命令更新 Supervisor 配置并启动进程：

```bash
sudo supervisorctl reread

sudo supervisorctl update

sudo supervisorctl start laravel-worker:*
```

有关 Supervisor 的更多信息，参阅 [Supervisor 文档](http://supervisord.org/index.html)。

## 处理失败的作业

有时候你的排队的作业会失败。别担心，事情并不总是按计划进行！Laravel 包含一种方便的方法来指定作业应该尝试的最大次数。作业超过此尝试次数后，它将被插入到 `failed_jobs` 数据库表中。要为 `failed_jobs` 表创建迁移，你可以使用 `queue:failed-table` 命令：

```bash
php artisan queue:failed-table

php artisan migrate
```

然后，在运行 [队列工作者](https://laravel.com/docs/5.8/queues#running-the-queue-worker) 时，你应该使用 `queue:work` 命令上的 `——tries` 开关指定尝试一个任务的最大次数。如果你没有为 `——tries` 选项指定一个值，将会无限期地尝试作业：

```bash
php artisan queue:work redis --tries=3
```

此外，可以使用 `——delay` 选项指定 Laravel 在重试失败的作业之前应该等待多少秒。默认情况下，作业会立即重试：

```bash
php artisan queue:work redis --tries=3 --delay=3
```

如果你希望在每个作业的基础上配置失败的作业重试延迟，你可以在你的排队的作业类上定义 `retryAfter` 属性来这样做：

```php
/**
 * 重试前等待的秒数。
 *
 * @var int
 */
public $retryAfter = 3;
```

### 作业失败后的清理工作

你可以直接在你的作业类上定义一个 `failed` 的方法，当发生失败时允许你去执行特定于作业的清理。这是向你的用户发送警告或还原作业执行的任何操作的最佳位置。导致作业失败的 `Exception` 将传递给 `failed` 方法：

```php
<?php

namespace App\Jobs;

use Exception;
use App\Podcast;
use App\AudioProcessor;
use Illuminate\Bus\Queueable;
use Illuminate\Queue\SerializesModels;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Contracts\Queue\ShouldQueue;

class ProcessPodcast implements ShouldQueue
{
    use InteractsWithQueue, Queueable, SerializesModels;

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
        // 处理上传的播客...
    }

    /**
     * 失败的作业处理。
     *
     * @param  Exception  $exception
     * @return void
     */
    public function failed(Exception $exception)
    {
        // 发送用户失败的通知等等...
    }
}
```

### 失败作业事件

如果你希望注册当作业失败时调用的事件，你可以使用 `Queue::failed` 方法。这是一个通过电子邮件或 [Slack](https://www.slack.com/) 通知你的团队的好机会。例如，我们可以从包含在 Laravel 中的 `AppServiceProvider` 中将回调附加到此事件上：

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Queue;
use Illuminate\Queue\Events\JobFailed;
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
        Queue::failing(function (JobFailed $event) {
            // $event->connectionName
            // $event->job
            // $event->exception
        });
    }
}
```

### 重试失败的作业

要查看已插入到你的 `failed_jobs` 数据库表中的你的所有失败作业，你可以使用 `queue:failed` Artisan 命令：

```bash
php artisan queue:failed
```

`queue:failed` 命令将列出作业 ID、连接、队列和失败时间。作业 ID 可用于重试失败的作业。例如，要重试 ID 为 `5` 的失败作业，请发出以下命令：

```bash
php artisan queue:retry 5
```

要重试你的所有失败作业，执行 `queue:retry` 命令并传递 `all` 作为其 ID：

```bash
php artisan queue:retry all
```

如果你想删除一个失败的作业，你可以使用 `queue:forget` 命令：

```bash
php artisan queue:forget 5
```

要删除你的所有失败的作业，你可以使用 `queue:flush` 命令：

```bash
php artisan queue:flush
```

### 忽略丢失的模型

当向作业注入一个 Eloquent 模型时，它会在被放入队列之前自动序列化，并在处理作业时恢复。但是，如果在作业等待工作者处理时删除了模型，那么你的作业可能会失败，出现 `ModelNotFoundException` 异常。

为了方便起见，你可以选择通过设置你的作业的 `deleteWhenMissingModels` 属性为 `true` 来自动删除缺失模型的作业：

```php
/**
 * 如果该作业的模型不再存在，则删除该作业。
 *
 * @var bool
 */
public $deleteWhenMissingModels = true;
```

## 作业事件

使用 `Queue` [外观](https://laravel.com/docs/5.8/facades) 上的 `before` 和 `after` 方法，你可以指定要在处理队列作业之前或之后执行的回调。这些回调是为仪表板执行额外日志记录或增量统计的好机会。通常，你应该从 [服务提供者](https://laravel.com/docs/5.8/providers) 调用这些方法。例如，我们可以使用 Laravel 中包含的 `AppServiceProvider`：

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Queue;
use Illuminate\Support\ServiceProvider;
use Illuminate\Queue\Events\JobProcessed;
use Illuminate\Queue\Events\JobProcessing;

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
        Queue::before(function (JobProcessing $event) {
            // $event->connectionName
            // $event->job
            // $event->job->payload()
        });

        Queue::after(function (JobProcessed $event) {
            // $event->connectionName
            // $event->job
            // $event->job->payload()
        });
    }
}
```

使用 `Queue` [外观](https://laravel.com/docs/5.8/facades) 上的 `looping` 方法，你可以指定在工作者尝试从队列获取作业之前执行的回调。例如，你可以注册一个闭包来回滚之前失败作业未关闭的任何事务：

```php
Queue::looping(function () {
    while (DB::transactionLevel() > 0) {
        DB::rollBack();
    }
});
```
