# 模仿

## 简介

在测试 Laravel 应用程序时，你可能希望『模仿』模拟你的应用程序的某些方面，以便在给定测试期间不会实际执行它们。例如，在测试调度事件的控制器时，你可能希望模拟事件监听器，以便在测试期间不会实际执行它们。这允许你仅测试控制器的 HTTP 响应，而不必担心事件监听器的执行，因为事件监听器可以在他们自己的测试用例中进行测试。

Laravel 为开箱即用为模仿事件，作业和外观提供帮手。这些助手主要提供了一个比 Mockery 更方便的层，因此你不必手动进行复杂的 Mockery 方法调用。你还可以使用 [Mockery](http://docs.mockery.io/en/latest/) 或 PHPUnit 创建自己的模仿或间谍。

## 模仿对象

当模仿的一个对象将通过 Laravel 的服务容器注入到你的应用程序的时，你需要将你的模仿的实例作为一个 `instance` 绑定绑定到容器中。这将指示容器使用对象的模仿实例，而不是构造对象本身：

```php
use Mockery;
use App\Service;

$this->instance(Service::class, Mockery::mock(Service::class, function ($mock) {
    $mock->shouldReceive('process')->once();
}));
```

***

为了使这个更加方便，你可以使用 `mock` 方法，该方法由 Laravel 的基本测试用例类提供：

```php
use App\Service;

$this->mock(Service::class, function ($mock) {
    $mock->shouldReceive('process')->once();
});
```

***

同样，如果你想窥探一个对象，Laravel 的基础测试用例类提供了一个 `spy` 方法作为 `Mockery::spy` 方法的一个方便的包装器：

```php
use App\Service;

$this->spy(Service::class, function ($mock) {
    $mock->shouldHaveReceived('process');
});
```

***

## 总线赝品

作为模仿的替代方法，你可以使用 `Bus` 外观的 `fake` 方法来阻止分配作业。使用 fakes 时，在执行测试代码后进行断言：

```php
<?php

namespace Tests\Feature;

use Tests\TestCase;
use App\Jobs\ShipOrder;
use Illuminate\Support\Facades\Bus;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Illuminate\Foundation\Testing\WithoutMiddleware;

class ExampleTest extends TestCase
{
    public function testOrderShipping()
    {
        Bus::fake();

        // 执行订单发货...

        Bus::assertDispatched(ShipOrder::class, function ($job) use ($order) {
            return $job->order->id === $order->id;
        });

        // 断言没有分配作业...
        Bus::assertNotDispatched(AnotherJob::class);
    }
}
```

***

## 事件赝品

## 邮件蠢哭
