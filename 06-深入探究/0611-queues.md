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

#### Redis

#### 其他驱动先决条件

## 创建作业

## 调度作业

## 排队闭包

## 运行队列工作者

## Supervisor 配置

## 处理失败的作业

## 作业事件
