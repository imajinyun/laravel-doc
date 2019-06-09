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

### 命令结构

### 闭包命令

## 定义输入的期望

## 命令 I/O

## 注册命令

## 以编程方式执行命令
