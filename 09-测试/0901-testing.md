# 测试：起步

## 简介

Laravel 的构建考虑了测试。实际上，开箱即用支持使用 PHPUnit 进行测试，并且已经为你的应用程序设置了 `phpunit.xml` 文件。框架还附带方便的帮助方法，允许你富于表现力地测试你的应用程序。

默认情况下，你的应用程序的 `tests` 目录包含两个目录：`Feature` 和 `Unit`。单元测试是专注于你的代码中非常小的，孤立的部分的测试。事实上，大多数单元测试可能只关注单一方法。功能测试可以测试你的代码的大部分，包括几个对象如何相互交互，甚至是对 JSON 端点的完整 HTTP 请求。

`Feature` 和 `Unit` 测试目录中都提供了 `ExampleTest.php` 文件。安装新的 Laravel 应用程序后，在命令行上运行 `phpunit` 以运行你的测试。

## 环境

当通过 `phpunit` 运行测试时，由于 `phpunit.xml` 文件中定义的环境变量，Laravel 会自动将配置环境设置为 `testing`。Laravel 还会在测试时自动配置会话和缓存到 `array` 驱动程序，这意味着在测试时会话或缓存数据不被持久化。

你可以根据需要自由定义其他测试环境配置值。可以在 `phpunit.xml` 文件中配置 `testing` 环境变量，但在运行你的测试之前，确保使用 `config：clear` Artisan 命令清除你的配置缓存！

此外，你可以在你的项目的根目录中创建 `.env.testing` 文件。运行 PHPUnit 测试或使用 `--env=testing` 选项执行 Artisan 命令时，此文件将覆盖 `.env` 文件。

## 创建 & 运行测试

要创建新的测试用例，请使用make：test Artisan命令：

```bash
// 在 Feature 目录中创建一个测试...
php artisan make:test UserTest

// 在 Unit 目录中创建一个测试...
php artisan make:test UserTest --unit
```

***

一旦生成了测试，你就可以像通常使用 PHPUnit 那样定义测试方法。要运行测试，从你的终端执行 `phpunit` 命令：

```php
<?php

namespace Tests\Unit;

use Tests\TestCase;
use Illuminate\Foundation\Testing\RefreshDatabase;

class ExampleTest extends TestCase
{
    /**
     * 一个基本测试实例。
     *
     * @return void
     */
    public function testBasicTest()
    {
        $this->assertTrue(true);
    }
}
```

{% hint style="danger" %}

如果在测试类中定义你自己的 `setUp` / `tearDown` 方法，确保在父类上调用各自的 `parent::setUp()` / `parent::tearDown()` 方法。

{% endhint %}
