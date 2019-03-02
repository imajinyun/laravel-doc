# 契约

## 简介

Laravel 的合约是一组接口，它定义了通过框架提供的核心服务。例如，一个 `Illuminate\Contracts\Queue\Queue` 合约定义了排队作业所需的方法，而 `Illuminate\Contracts\Mail\Mailer` 合约定义了发送电子邮件所需的方法。

每个合约有一个框架提供的相应实现。例如，Laravel 提供各种驱动程序的队列实现，以及一个由 [SwiftMailer](https://swiftmailer.symfony.com/) 提供支持的邮件程序实现。

所有的 Laravel 合约都存在于 [它们自己的 GitHub 仓库](https://github.com/illuminate/contracts)。这为所有可用的合约以及包开发者可能使用到的单个、解耦的包提供了快速参考点。

### 合约 VS Facades

Laravel 的 [facades](https://laravel.com/docs/5.8/facades) 和助手函数提供一个简单的方式来利用 Laravel 服务，而无需从服务容器中键入类型提示和解析合约。在大多数情况下，每个 facade 都有等效的合约。

与 facades 不同，不要求你在类的构造方法中要求它们，合约允许你为类定义显式的依赖项。一些开发者偏好去显式地定义他们的依赖关系这种方式，并因此更偏爱使用合约，而其他开发人员则喜欢使用 facades。

{% hint style="info" %}

无论你偏爱 facades 或合约，大多数应用程序都可以。但是，如果你构建一个包，你应该强烈地考虑使用合约，因为它们将在包的上下文中很容易测试。

{% endhint %}

## 何时使用契约

正如其它地方所讨论的那样，使用合约还是 Facades 很大程度上取决于你个人品味或开发团队的偏好。合约和 Facades 都可用来构建健壮的、良好测试的 Laravel 应用程序。只要你保持你的类的职责单一，你会发现使用合约和 Facades 的实际差异是很小的。

然而，你仍然可能有几个关于合约的问题。例如，为什么要使用接口？用接口不是更复杂吗？让我们提炼如下标题对于使用接口的原因：松耦合和简单。

### 松耦合

首先，让我们回顾一些与缓存实现紧密耦合的代码。考虑以下几点：

```php
<?php

namespace App\Orders;

class Repository
{
    /**
     * 缓存实例。
     */
    protected $cache;

    /**
     * 创建一个新的存储实例。
     *
     * @param  \SomePackage\Cache\Memcached  $cache
     * @return void
     */
    public function __construct(\SomePackage\Cache\Memcached $cache)
    {
        $this->cache = $cache;
    }

    /**
     * 通过 ID 检索订单。
     *
     * @param  int  $id
     * @return Order
     */
    public function find($id)
    {
        if ($this->cache->has($id))    {
            //
        }
    }
}
```

在这个类中，代码紧密耦合到给定的缓存实现。它紧密耦合是因为我们依赖于包供应商的一个具体缓存类。如果该包的 API 发生改变，我们的代码也必须更改。

同样，如果我们想用另一种的技术（Redis）替换我们的底层缓存技术（Memcached），我们将再次修改我们的存储库。我们的存储库不应该对谁提供数据或者如何提供数据有太多的了解。

**我们可以通过依赖于简单的，与供应商无关的接口来改进我们的代码，而不是之前的方法：**

```php
<?php

namespace App\Orders;

use Illuminate\Contracts\Cache\Repository as Cache;

class Repository
{
    /**
     * 缓存实例。
     */
    protected $cache;

    /**
     * 创建一个新的存储实例。
     *
     * @param  Cache  $cache
     * @return void
     */
    public function __construct(Cache $cache)
    {
        $this->cache = $cache;
    }
}
```

现在代码没有耦合到任何特定的供应商，甚至 Laravel。由于合约包不包含实现和依赖关系，你可以轻松编写任何给定合约的替代实现，允许你去替换你的缓存实现无需修改任何缓存消费代码。

### 简单

## 如何使用契约

## 契约参考
