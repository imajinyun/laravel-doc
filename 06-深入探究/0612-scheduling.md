# 任务计划

## 简介

过去，你可能在你的服务器上需要计划的每项任务生成了一个 Cron 条目。但是，这很快就会变得很麻烦，因为你的任务计划不再处于源代码管理中，你必须通过 SSH 连接到你的服务器以添加其他 Cron 条目。

Laravel 的命令计划器允许你在 Laravel 本身内流畅并有表现地定义你的命令计划。使用计划程序时，你的服务器上只需要一个 Cron 条目。你的任务计划在 `app/Console/Kernel.php` 文件的 `schedule` 方法中定义。为了帮助你入门，在方法中定义了一个简单的示例。

### 启动计划器

在使用调度程序时，你只需要将以下 Cron 条目添加到你的服务器。如果你不知道如何向你的服务器添加 Cron 条目，考虑使用 [Laravel Forge](https://forge.laravel.com/) 这样的服务，它可以为你管理 Cron 条目：

```bash
* * * * * cd /path-to-your-project && php artisan schedule:run >> /dev/null 2>&1
```

这个 Cron 将每分钟调用 Laravel 命令计划器。当 `schedule:run` 命令执行时，Laravel 将评估你的计划的任务并运行到期的任务。

## 定义计划

你可以在 `App\Console\Kernel` 类的 `schedule` 方法中定义你的所有计划的任务。首先，让我们看一个计划任务的例子。在本例中，我们将计划一个每天午夜调用的 `Closure`。在 `Closure` 中，我们将执行一个数据库查询来清除一个表：

```php
<?php

namespace App\Console;

use Illuminate\Support\Facades\DB;
use Illuminate\Console\Scheduling\Schedule;
use Illuminate\Foundation\Console\Kernel as ConsoleKernel;

class Kernel extends ConsoleKernel
{
    /**
     * 你的应用程序提供的 Artisan 命令。
     *
     * @var array
     */
    protected $commands = [
        //
    ];

    /**
     * 定义应用程序的命令计划。
     *
     * @param  \Illuminate\Console\Scheduling\Schedule  $schedule
     * @return void
     */
    protected function schedule(Schedule $schedule)
    {
        $schedule->call(function () {
            DB::table('recent_users')->delete();
        })->daily();
    }
}
```

除了使用闭包进行计划外，还可以使用 [可调用对象](https://secure.php.net/manual/en/language.oop5.magic.php#object.invoke)。可调用对象是包含 `__invoke` 方法的简单 PHP 类：

```php
$schedule->call(new DeleteRecentUsers)->daily();
```

### 计划 Artisan 命名

除了计划闭包调用，你还可以计划 [Artisan 命令](https://laravel.com/docs/5.8/artisan) 和操作系统命令。例如，你可以使用 `command` 方法使用命令的名称或类来计划 Artisan 命令：

```php
$schedule->command('emails:send Taylor --force')->daily();

$schedule->command(EmailsCommand::class, ['Taylor', '--force'])->daily();
```

### 计划排队的作业

`job` 方法可用于计划一个 [排队的作业](https://laravel.com/docs/5.8/queues)。此方法提供了一种方便的方法来计划作业，而无需使用 `call` 方法手动创建闭包来对作业进行排队：

```php
$schedule->job(new Heartbeat)->everyFiveMinutes();

// 将作业调度到『heartbeats』队列...
$schedule->job(new Heartbeat, 'heartbeats')->everyFiveMinutes();
```

### 计划 Shell 命令

`exec` 方法可用于向操作系统发出命令：

```php
$schedule->exec('node /home/forge/script.js')->daily();
```

### 计划频率选项

你可以为你的任务分配各种各样的时间表：

| 方法 | 描述 |
| --- | --- |
| `->cron('* * * * *');` | 按照自定义 Cron 计划运行任务 |
| `->everyMinute();` | 每分钟运行任务 |
| `->everyFiveMinutes();` | 每五分钟运行任务 |
| `->everyTenMinutes();` | 每十分钟运行任务 |
| `->everyFifteenMinutes();` | 每十五分钟运行任务 |
| `->everyThirtyMinutes();` | 每三十分钟运行任务 |
| `->hourly();` | 每小时运行任务 |
| `->hourlyAt(17);` | 每小时 17 分钟运行任务 |
| `->daily();` | 每天午夜执行任务 |
| `->dailyAt('13:00');` | 每天 13:00 运行任务 |
| `->twiceDaily(1, 13);` | 每天 1 点和 13:00 运行任务 |
| `->weekly();` | 每周运行任务 |
| `->weeklyOn(1, '8:00');` | 每周星期一早上 8 点运行任务 |
| `->monthly();` | 每月运行任务 |
| `->monthlyOn(4, '15:00');` | 每月 4 号 15:00 运行任务 |
| `->quarterly();` | 每季度运行任务 |
| `->yearly();` | 每年运行任务 |
| `->timezone('America/New_York');` | 设置时区 |

这些方法可以与其他约束相结合，以创建更精细调整的计划，这些计划仅在一周的特定日子运行。例如，要计划命令在每周的星期一运行：

```php
// 每周一下午 1 点运行一次...
$schedule->call(function () {
    //
})->weekly()->mondays()->at('13:00');

// 工作日从上午 8 点到下午 5 点每小时运行...
$schedule->command('foo')
          ->weekdays()
          ->hourly()
          ->timezone('America/Chicago')
          ->between('8:00', '17:00');
```

下面是附加的日程约束列表：

| 方法 | 描述 |
| `->weekdays();` | 任务限制为工作日 |
| `->weekends();` | 任务限制为周末 |
| `->sundays();` | 任务限制为星期日 |
| `->mondays();` | 任务限制为星期一 |
| `->tuesdays();` | 任务限制为星期二 |
| `->wednesdays();` | 任务限制为星期三 |
| `->thursdays();` | 任务限制为星期四 |
| `->fridays();` | 任务限制为星期五 |
| `->saturdays();` | 任务限制为星期六 |
| `->between($start, $end);` | 限制任务在开始和结束时间之间运行 |
| `->when(Closure);` | 根据真值测试来限制任务 |
| `->environments($env);` | 将任务限制在特定环境中 |

#### 在时间约束之间

#### 真值测试约束

#### 环境约束

### 时区

### 防止任务重叠

### 一台服务器上运行任务

### 后台任务

### 维护模式

## 任务输出

## 任务钩子
