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

## 任务输出

## 任务钩子
