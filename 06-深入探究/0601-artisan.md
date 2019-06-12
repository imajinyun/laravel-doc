# Artisan 控制台

## 简介

Artisan 是 Laravel 中包含的命令行界面。它提供了许多有用的命令，可以帮助你构建应用程序。要查看所有可用的 Artisan 命令的列表，可以使用 `list` 命令：

```php
php artisan list
```

每个命令还包括一个『帮助』屏幕，用于显示和描述命令的可用参数和选项。要查看帮助屏幕，请在命令名称前加上 `help`：

```bash
php artisan help migrate
```

### Tinker (REPL)

所有 Laravel 应用程序都包含 Tinker，这是一个由 [PsySH](https://github.com/bobthecow/psysh) 包驱动的 REPL。Tinker 允许你在命令行上与整个 Laravel 应用程序交互，包括 Eloquent ORM、作业、事件等等。要进入 Tinker 环境，请运行 `tinker` Artisan 命令：

```php
php artisan tinker
```

你可以使用 `vendor:publish` 命令发布 Tinker 的配置文件：

```bash
php artisan vendor:publish --provider="Laravel\Tinker\TinkerServiceProvider"
```

#### 命令白名单

Tinker 使用一个白列表来确定哪些 Artisan 命令可以在它的 shell 中运行。默认情况下，你可以运行 `clear-compile`、`down`、`env`、`inspire`、`migration`、`optimization` 和 `up` 命令。如果你想列出更多的命令，可以将它们添加到你的 `tinker.php` 配置文件中的 `commands` 数组中：

```php
'commands' => [
    // App\Console\Commands\ExampleCommand::class,
],
```

#### 别名黑名单

通常，Tinker 会随着你在 Tinker 中引入的类自动别名。但是，你可能希望永远不要别名某些类。您可以通过在 `tinker.php` 配置文件的 `don alias` 数组中列出类来实现这一点：

```php
'dont_alias' => [
    App\User::class,
],
```

## 编写命令

除了 Artisan 提供的命令之外，你还可以构建自己的自定义命令。命令通常存储在 `app/Console/Commands` 目录中；但是，只要你的命令被 Composer 加载，你就可以自由选择自己的存储位置。

### 生成命令

要创建新命令，请使用 `make:command` Artisan 命令。此命令将在 `app/Console/Commands` 目录中创建一个新的命令类。如果你的应用程序中不存在此目录，请不要担心，因为它将在你第一次运行 `make:command` Artisan 命令时创建。生成的命令将包括所有命令上存在的默认属性和方法集：

```php
php artisan make:command SendEmails
```

### 命令结构

生成命令之后，应该填写该类的 `signature` 和 `description` 属性，这些属性将用于在 `list` 屏幕上显示你的命令。在执行命令时将调用 `handle` 方法。你可以将命令逻辑放在此方法中。

{% hint style="info" %}

为了实现更大的代码重用，最好让控制台命令保持轻量，并让它们遵从应用程序服务来完成他们的任务。在下面的示例中，请注意，我们注入了一个服务类来完成发送电子邮件的『繁重工作』。

{% endhint %}

让我们看一个示例命令。注意，我们能够将需要的任何依赖项注入到命令的 `handle` 方法中。Laravel [服务容器](https://laravel.com/docs/5.8/container) 将自动注入此方法的签名中类型提示的所有依赖项：

```php
<?php

namespace App\Console\Commands;

use App\User;
use App\DripEmailer;
use Illuminate\Console\Command;

class SendEmails extends Command
{
    /**
     * 控制台命令的名称和签名。
     *
     * @var string
     */
    protected $signature = 'email:send {user}';

    /**
     * 控制台命令描述。
     *
     * @var string
     */
    protected $description = 'Send drip e-mails to a user';

    /**
     * 创建一个新的命令实例。
     *
     * @return void
     */
    public function __construct()
    {
        parent::__construct();
    }

    /**
     * 执行控制台命令。
     *
     * @param  \App\DripEmailer  $drip
     * @return mixed
     */
    public function handle(DripEmailer $drip)
    {
        $drip->send(User::find($this->argument('user')));
    }
}
```

### 闭包命令

基于闭包的命令提供了将控制台命令定义为类的替代方法。与路由闭包是控制器的替代方法相同，可以将命令闭包视为命令类的替代方法。在 `app/Console/Kernel.php` 文件的 `commands` 方法中，Laravel 加载 `routes/console.php` 文件：

```php
/**
 * 为应用程序注册基于闭包的命令。
 *
 * @return void
 */
protected function commands()
{
    require base_path('routes/console.php');
}
```

尽管这个文件没有定义 HTTP 路由，但是它定义了应用程序中基于控制台的入口点（路由）。在这个文件中，你可以使用 `Artisan::command` 方法定义所有基于闭包的路由。`command` 方法接受两个参数：[命令签名](https://laravel.com/docs/5.8/artisan#defining-input-expectations) 和一个接收命令参数和选项的闭包：

```php
Artisan::command('build {project}', function ($project) {
    $this->info("Building {$project}!");
});
```

闭包绑定到底层命令实例，因此你可以完全访问通常能够在完整命令类上访问的所有助手方法。

#### 类型提示依赖

除了接收你的命令的参数和选项外，命令闭包还可以类型提示你希望从 [服务容器](https://laravel.com/docs/5.8/container) 中解析的其他依赖项：

```php
use App\User;
use App\DripEmailer;

Artisan::command('email:send {user}', function (DripEmailer $drip, $user) {
    $drip->send(User::find($user));
});
```

#### 闭包命令描述

在定义基于闭包的命令时，可以使用 `description` 方法向命令添加描述。当你运行 `php artisan list` 或 `php artisan help` 命令时，将显示此描述：

```php
Artisan::command('build {project}', function ($project) {
    $this->info("Building {$project}!");
})->describe('Build the project');
```

## 定义输入的期望

在编写控制台命令时，通常通过参数或选项从用户收集输入。Laravel 使你可以非常方便地使用命令上的 `signature` 属性来定义你期望从用户得到的输入。`signature` 属性允许你用一种单一的、有表现力的、类似于路由的语法来定义命令的名称、参数和选项。

### 参数

所有用户提供的参数和选项都用大括号括起来。在下面的示例中，该命令定义了一个 **必须** 的参数：`user`：

```php
/**
 * 控制台命令的名称和签名。
 *
 * @var string
 */
protected $signature = 'email:send {user}';
```

你还可以使参数成为可选的，并为参数定义默认值：

```php
// 可选的参数...
email:send {user?}

// 具有默认值的可选参数...
email:send {user=foo}
```

### 选项

选项和参数一样，是用户输入的另一种形式。选项在命令行上指定时，以两个连字符（`--`）作为前缀。有两种类型的选项：接收值的选项和不接收值的选项。不接受值的选项充当布尔『开关』。让我们看一个这类选项的例子:

```php
/**
 * 控制台命令的名称和签名。
 *
 * @var string
 */
protected $signature = 'email:send {user} {--queue}';
```

在这个实例中，当调用 Artisan 命令时 `--queue` 开关可以被指定。如果 `--queue` 开关被传递，选项的值将为 `true`。否则，值将为 `false`：

```bash
php artisan email:send 1 --queue
```

#### 带有值的选项

接下来，让我们看看一个期望值的选项。如果用户必须为某个选项指定一个值，请在选项名称后面加上（`=`）符号：

```php
/**
 * 控制台命令的名称和签名。
 *
 * @var string
 */
protected $signature = 'email:send {user} {--queue=}';
```

在这个实例中，用户可以像这样传递一个值：

```php
php artisan email:send 1 --queue=default
```

你可以通过在选项名称后面指定默认值来为选项分配默认值。如果用户没有传递选项值，则使用默认值：

```bash
email:send {user} {--queue=default}
```

#### 选项快捷方式

要在定义选项时分配快捷方式，可以在选项名称之前指定它，并使用 `|` 分隔符将快捷方式与完整的选项名称分隔：

```bash
email:send {user} {--Q|queue}
```

### 输入数组

如果希望定义期望数组输入的参数或选项，可以使用 `*` 字符。首先，让我们看一个指定数组参数的示例：

```bash
email:send {user*}
```

调用此方法时，可以将 `user` 参数传递到命令行。例如，下面的命令将 `user` 的值设置为 `['foo', 'bar']`：

```bash
php artisan email:send foo bar
```

当定义期望数组输入的选项时，传递给命令的每个选项值都应该以选项名作为前缀：

```bash
email:send {user} {--id=*}

php artisan email:send --id=1 --id=2
```

### 输入描述

你可以通过使用冒号将参数从描述中分隔开来，将描述分配给输入参数和选项。如果你需要一点额外的空间来定义你的命令，请随意将扩展定义跨越多行：

```php
/**
 * 控制台命令的名称和签名。
 *
 * @var string
 */
protected $signature = 'email:send
                        {user : The ID of the user}
                        {--queue= : Whether the job should be queued}';
```

## 命令 I/O

### 检索输入

在执行命令时，很显然你将需要去访问命令接受的参数和选项的值。为此，可以使用 `argument` 和 `option` 方法：

```php
/**
 * 执行控制台命令。
 *
 * @return mixed
 */
public function handle()
{
    $userId = $this->argument('user');

    //
}
```

如果你需要检索所有参数作为一个 `array`，调用 `arguments` 方法：

```php
$arguments = $this->arguments();
```

使用 `option` 方法可以像检索参数一样轻松地检索选项。要以数组的形式检索所有选项，请调用 `options` 方法：

```php
// 检索一个指定的选项...
$queueName = $this->option('queue');

// 检索所有选项...
$options = $this->options();
```

如果参数或选项不存在，则返回 `null`。

### 提示输入

除了显示输出，你还可以要求用户在执行命令期间提供输入。`ask` 方法将用给定的问题提示用户，接受他们的输入，然后将用户的输入返回给你的命令：

```php
/**
 * 执行的控制台命令。
 *
 * @return mixed
 */
public function handle()
{
    $name = $this->ask('What is your name?');
}
```

`secret` 方法类似于 `ask`，但是当用户在控制台中键入时，用户的输入将不可见。这种方法在询问敏感信息（如密码）时非常有用：

```php
$password = $this->secret('What is the password?');
```

#### 要求确认

如果需要向用户请求简单的确认，可以使用 `confirm` 方法。默认情况下，该方法将返回 `false`。但是，如果用户响应提示输入 `y` 或 `yes`，该方法将返回 `true`。

```php
if ($this->confirm('Do you wish to continue?')) {
    //
}
```

#### 自动完成

`anticipate` 方法可用于为可能的选择提供自动完成。无论自动完成提示是什么，用户仍然可以选择任何答案：

```php
$name = $this->anticipate('What is your name?', ['Taylor', 'Dayle']);
```

#### 多项选择题

如果需要为用户提供一组预定义的选项，可以使用 `choice` 方法。如果没有选择任何选项，可以将默认值的数组索引设置为返回：

```php
$name = $this->choice('What is your name?', ['Taylor', 'Dayle'], $defaultIndex);
```

### 编写输出

要将输出发送到控制台，要使用 `line`，`info`，`comment`，`question` 和 `error` 方法。每一种方法都将为他们的意图使用适当的 ANSI 颜色。例如，让我们向用户显示一些一般信息。通常，`info` 方法将在控制台中显示为绿色文本：

```php
/**
 * 执行控制台命令。
 *
 * @return mixed
 */
public function handle()
{
    $this->info('Display this on the screen');
}
```

要显示错误消息，使用 `error` 方法。错误消息文本通常显示为红色：

```php
$this->error('Something went wrong!');
```

如果你想显示没有颜色的控制台输出纯文本，使用 `line` 方法：

```php
$this->line('Display this on the screen');
```

#### 表格布局

`table` 方法可以轻松地正确格式化多行 / 多列数据。只需将标题和行传递给方法即可。宽度和高度将基于给定的数据动态计算：

```php
$headers = ['Name', 'Email'];

$users = App\User::all(['name', 'email'])->toArray();

$this->table($headers, $users);
```

#### 进度条

对于长时间运行的任务，显示进度指示器可能很有帮助。使用输出对象，我们可以启动、前进和停止进度条。首先，定义流程将迭代的总步骤数。然后，推进进度条之后再处理每个条目：

```php
$users = App\User::all();

$bar = $this->output->createProgressBar(count($users));

$bar->start();

foreach ($users as $user) {
    $this->performTask($user);

    $bar->advance();
}

$bar->finish();
```

更多高级选项，查阅 [Symfony Process Bar 组件 文档](https://symfony.com/doc/current/components/console/helpers/progressbar.html)。

## 注册命令

## 以编程方式执行命令
