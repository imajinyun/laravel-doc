# Valet

* [简介](#jian-jie)
  * [Valet 还是 Homestead](#valet-hai-shi-homestead)
* [安装](#an-zhuang)
  * [其它](#qi-ta)
  * [升级](#sheng-ji)
* [服务站点](#fu-wu-zhan-dian)
  * [park 命令](#park-ming-ling)
  * [link 命令](#link-ming-ling)
  * [使用 TLS 保护站点](#shi-yong-tls-bao-hu-zhan-dian)
* [共享站点](#gong-xiang-zhan-dian)
* [自定义 Valet 驱动](#zi-ding-yi-valet-qu-dong)
  * [本地驱动](#ben-di-qu-dong)
* [其它 Valet 命令](#qi-ta-valet-ming-ling)

## 简介

Valet 是适用于 Mac 极简主义者的 Laravel 开发环境。没有 Vagrant，没有 `/etc/hosts` 文件。你甚至可以使用本地通道公开共享你的站点。是的，我们也喜欢它。

Laravel Valet 配置你的 Mac 在机器启动时总是在后台运行 [Nginx](https://www.nginx.com/)。然后，使用 [DnsMasq](https://en.wikipedia.org/wiki/Dnsmasq)，Valet 代理将 `*.test` 域上的所有请求指向到你的本地机器上安装的站点。

换句话说，一个超快的 Laravel 开发环境，使用了大约 7 MB 的 RAM。Valet 不是 Vagrant 或 Homestead 的完成替代品，但如果你想要灵活的基础，偏好极快的速度，或在一个受限数量的 RAM 机器上工作时，它提供了一个很好的选择。

开箱即用的 Valet 支持的，包含并不仅限于：

* [Laravel](https://laravel.com/)
* [Lumen](https://lumen.laravel.com/)
* [Bedrock](https://roots.io/bedrock/)
* [CakePHP 3](https://cakephp.org/)
* [Concrete5](https://www.concrete5.org/)
* [Contao](https://contao.org/en/)
* [Craft](https://craftcms.com/)
* [Drupal](https://www.drupal.org/)
* [Jigsaw](https://jigsaw.tighten.co/)
* [Joomla](https://www.joomla.org/)
* [Katana](https://github.com/themsaid/katana)
* [Kirby](https://getkirby.com/)
* [Magento](https://magento.com/)
* [OctoberCMS](https://octobercms.com/)
* [Sculpin](https://sculpin.io/)
* [Slim](https://www.slimframework.com/)
* [Statamic](https://statamic.com/)
* Static HTML
* [Symfony](https://symfony.com/)
* [WordPress](https://wordpress.org/)
* [Zend](https://framework.zend.com/)

不管怎样，你可以合适你自己 [自定义的驱动程序](https://laravel.com/docs/5.8/valet#custom-valet-drivers) 扩展 Valet。

### Valet 还是 Homestead

如你所知，Laravel 提供另一个本地 Laravel 开发环境 [Homestead](https://laravel.com/docs/5.8/homestead)。Homestead 和 Valet 对关于他们的目标受众和他们对本地开发的方法上有所不同。Homestead 提供了一个具有自动 Nginx 配置的完整 Ubuntu 虚拟机。如果你想要一个完全虚拟化的 Linux 开发环境或者在 Windows / Linux 系统上，Homestead 是一个极好的选择。

Valet 仅支持 Mac，并要求你直接在你的本地机器上安装 PHP 和数据库服务器。这通过 [Homebrew](https://brew.sh/) 的 `brew install php` 和 `brew install mysql` 很容易实现。Valet 以极少的资源消耗提供一个极速的本地开发环境，因此对于仅仅需要 PHP / MySQL 且不需要完全虚拟化开发环境的开发者是极好的。

Valet 和 Homestead 都是配置你的 Laravel 开发环境的绝佳选择。选择哪一个将取决你的个人品味和你的团队需要。

## 安装

**Valet 需要 masOS 和 [Homebrew](https://brew.sh/)。安装之前，你应该确保没有其它程序，比如 Apache 或 Nginx 绑定你的本地机器的 80 端口。**

* 安装或使用 `brew update` 升级 [Homebrew](https://brew.sh/) 到最新的版本。
* 使用 Homebrew 的 `brew install php` 安装 PHP 7.3。
* 安装 [Composer](https://getcomposer.org/)。
* 通过 Composer 的 `composer global require laravel/valet` 安装 Valet。确保 `~/.composer/vendor/bin` 目录在你的系统的『PATH』环境变量中。
* 运行 `valet install` 命令。这将配置和安装 Valet 和 DnsMasq，并注册 Valet 的守护进程以在你的系统启动时运行。

一旦 Valet 被安装，使用比如 `ping foobar.test` 这样的命令在你的终端尝试 ping 任何 `*.test` 域名。如果 Valet 安装正确，你应当看到此域名在 `127.0.0.1` 上响应。

每次你的机器启动，Valet 将自动启动其守护进程。初始化 Valet 安装一旦完成就无需再次运行 `valet start` 或 `valet install` 命令。

### 其它

#### 使用其它域名

默认情况下，Valet 使用 `.test` 的顶级域名为你的项目服务。如果你想使用另一个域名，你可以使用 `valet tld tld-name` 命令。

例如，如果你想使用 `.app` 代替 `.test`，运行 `valet tld app`，Valet 将自动开始以 `*.app` 服务你的项目。

#### 数据库

如果你需要数据库，在你的命令行运行 `brew install mysql@5.7` 来尝试安装 MySQL。一旦 MySQL 被安装，你可以使用 `brew services start mysql@5.7` 命令来启动它。然后你可以使用 `root` 用户名和空的密码字符串连接到 `127.0.0.1` 的数据库。

#### PHP 版本

Valet 允许你使用 `valet use php@version` 命令去切换 PHP 版本。Valet 将通过 Brew 安装指定的 PHP 版本，如果它还没有安装时：

```bash
valet use php@7.2

valet use php
```

### 升级

你可以在你的终端使用 `composer global update` 命令更新你的 Valet 安装。升级之后，运行 `valet install` 命令是一个好的实践，以便 Valet 可以根据需要对配置文件进行额外升级。

#### 升级到 Valet 2.0

Valet 2.0 将 Valet 的底层 web 服务器从 Caddy 过渡到 Nginx。升级到这个版本之前，你应该运行如下的命令去停止和卸载现有的 Caddy 守护进程。

```bash
valet stop
valet uninstall
```

接下来，你应当升级到 Valet 的最新版本。根据你安装 Valet 的方式，这个通常通过 Git 或 Composer 来完成。如果你通过 Composer 安装了 Valet，你应当使用如下的命令去更新到最新的主版本：

```bash
composer global require laravel/valet
```

一旦全新的 Valet 源码被下载后，你应当运行 `install` 命令：

```bash
valet install
valet restart
```

升级过后，可能需要重新设置或重新链接你的站点。

## 服务站点

Valet 一旦被安装，你就可以开始服务站点了。Valet 提供两个命令来帮助你为你的 Laravel 站点提供服务：`park` 和 `link`。

### park 命令

* 通过运行 `mkdir ~/Sites` 之类的命令在你的 Mac 上创建一个新目录。接下来，`cd ~/Sites` 并运行 `valet park`。此命令将你的当前工作目录注册为 Valet 应当搜索站点的一个路径。
* 接下来，在这个目录创建一个新的 Laravel 站点：`laravel new blog`。
* 在浏览器打开 `http://blog.test`。

就这些了。现在，你在『parked』的目录中创建的任何 Laravel 项目都将自动使用 `http://folder-name.test` 约定提供服务。

### link 命令

`link` 命令也可用于为 Laravel 站点提供服务。如果你想在一个目录而不是整个目录里服务单个站点，这个命令将很有用。

* 要使用这个命令，导航到你的其中一个项目并在终端运行 `valet link app-name` 命令。Valet 将在 `~/.config/valet/Sites` 下创建一个符号链接，此符号链接指向你当前的工作目录。
* 运行 `link` 命令之后，你能在浏览器输入 `http://app-name.test` 访问站点。

运行 `valet links` 命令去查看所有链接的目录列表。你可以使用 `valet unlink app-name` 去销毁符号链接。

{% hint style="danger" %}

你可以使用 `valet link` 从多个（子）域为相同的项目提供服务。要将子域或其它域名添加到项目，从项目目录中运行 `valet link subdomain.app-name` 命令。

{% endhint %}

### 使用 TLS 保护站点

默认情况下，Valet 服务站点通过纯 HTTP 的方式。但是，如果你想一个站点使用 HTTP/2 通过加密 TLS 提供服务，使用 `secure` 命令。例如，如果你的站点通过 Valet 在 `laravel.test` 域上提供服务，你应当运行如下的命令去保护它：

```bash
valet secure laravel
```

要一个站点『解除保护』并恢复为通过纯 HTTP 提供服务它的流量，使用 `unsecure` 命令。与 `secure` 命令一样，该命令接受你希望去解除保护的主机名称：

```bash
valet unsecure laravel
```

## 共享站点

Valet 甚至包括与世界共享你的本地站点的命令。一旦安装 Valet 后无需安装其它软件。

要共享站点，在终端中导航你的站点目录并运行 `valet share` 命令。一个公开可访问的 URL 将被插入到剪贴板中，并可以直接粘贴到你的浏览器中。仅此而已。

要停止共享你的站点，请按 `Control + C` 去取消该过程。

## 自定义 Valet 驱动

你可以编写你自己的 Valet『驱动』来为 Valet 原本不支持的其它框架或 CMS 上运行的 PHP 应用程序提供服务。当你安装 Valet 时，会创建一个包含 `SampleValetDriver.php` 文件的 `~/.config/valet/Drivers` 目录。该文件包含一个示例驱动程序实现，演示了如何编写一个自定义的驱动程序。编写驱动程序仅需要你去实现三个方法：`serves`，`isStaticFile` 和 `frontControllerPath`。

这三个方法都接受 `$sitePath`，`$siteName` 和 `$uri` 值作为参数。`$sitePath` 是你的机器上提供站点的完全限定路径。比如 `/Users/Lisa/Sites/my-project`。`$siteName` 是域（`my-project`）的『主机』/『站点名称』部分。`$uri` 是即将到来的请求 URL（/foo/bar）。

一旦你完成你的自定义 Valet 驱动，使用 `FrameworkValetDriver.php` 命名约定将它放置在 `~/.config/valet/Drivers` 目录中。例如，如果你为 WordPress 编写了一个自定义的 valet 驱动，你的文件名称应该是 `WordPressValetDriver.php`。

我们来看看自定义的 Valet 驱动程序应该实现的每种方法的示例实现。

### serves 方法

如果你的驱动程序应当处理即将到来的请求时， `serves` 方法应该返回 `true`。否则，此方法应当返回 `false`。因此，在此方法中，你应该你企图确定给定的 `$sitePath` 是否包含你尝试提供的类型的项目。

例如，让我们假装正在编写一个 `WordPressValetDriver`。我们的 `serves` 方法可能看起来如下所示：

```php
/**
 * 确定驱动程序是否满足请求。
 *
 * @param  string  $sitePath
 * @param  string  $siteName
 * @param  string  $uri
 * @return bool
 */
public function serves($sitePath, $siteName, $uri)
{
    return is_dir($sitePath.'/wp-admin');
}
```

### isStaticFile 方法

`isStaticFile` 应当确定即将到来的请求是否针对一个『静态』文件，比如：图片和样式表。如果文件是静态的，此方法应当返回静态文件在磁盘上的完全限定路径。如果即将到来的请求不是针对一个静态文件，这个方法应当返回 `false`：

```php
/**
 * 确定即将到来的请求是否针对静态文件。
 *
 * @param  string  $sitePath
 * @param  string  $siteName
 * @param  string  $uri
 * @return string|false
 */
public function isStaticFile($sitePath, $siteName, $uri)
{
    if (file_exists($staticFilePath = $sitePath.'/public/'.$uri)) {
        return $staticFilePath;
    }

    return false;
}
```

{% hint style="danger" %}

如果 `serves` 方法对即将到来的请求且请求 URI 不是 `/` 返回 `true` 时，才会调用 `isStaticFile` 方法。

{% endhint %}

### frontControllerPath 方法

`frontControllerPath` 方法应当返回你的应用程序的『前端控制器』完全限定的路径，它通常是你的 『index.php』文件或者等效的文件：

```php
/**
 * 获取对应用程序的前端控制器的完全解析路径。
 *
 * @param  string  $sitePath
 * @param  string  $siteName
 * @param  string  $uri
 * @return string
 */
public function frontControllerPath($sitePath, $siteName, $uri)
{
    return $sitePath.'/public/index.php';
}
```

### 本地驱动

如果你想为单个应用程序自定义 Valet 驱动程序，在应用程序的根目录下创建一个 `LocalValetDriver.php` 文件。你的自定义驱动程序可以扩展 `ValetDriver` 基类或者继承一个现有的应用程序的特定驱动程序，比如：`LaravelValetDriver`：

```php
class LocalValetDriver extends LaravelValetDriver
{
    /**
     * 确定驱动程序是否满足请求。
     *
     * @param  string  $sitePath
     * @param  string  $siteName
     * @param  string  $uri
     * @return bool
     */
    public function serves($sitePath, $siteName, $uri)
    {
        return true;
    }

    /**
     * 获取对应用程序的前端控制器的完全解析路径。
     *
     * @param  string  $sitePath
     * @param  string  $siteName
     * @param  string  $uri
     * @return string
     */
    public function frontControllerPath($sitePath, $siteName, $uri)
    {
        return $sitePath.'/public_html/index.php';
    }
}
```

## 其它 Valet 命令

| 命令              | 描述                                                   |
| :---------------- | :----------------------------------------------------- |
| `valet forget`    | 从一个『驻留』目录运行此命令，从驻留目录列表将其它移除 |
| `valet paths`     | 查看所有『驻留』路径                                   |
| `valet restart`   | 重启 Valet 守护进程                                    |
| `valet start`     | 开启 Valet 守护进程                                    |
| `valet stop`      | 停止 Valet 守护进程                                    |
| `valet uninstall` | 完成卸载 Valet 守护进程                                |
