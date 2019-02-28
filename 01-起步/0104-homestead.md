# Homestead

* [简介](#jian-jie)
* [安装与设置](#an-zhuang-yu-she-zhi)
  * [起步](#qi-bu)
  * [配置 Homestead](#pei-zhi-homestead)
  * [运行 Vagrant 盒子](#yun-xing-vagrant-he-zi)
  * [每个项目中安装](#mei-ge-xiang-mu-zhong-an-zhuang)
  * [安装 MariaDB](#an-zhuang-mariadb)
  * [安装 MongoDB](#an-zhuang-mongodb)
  * [安装 Elasticsearch](#an-zhuang-elasticsearch)
  * [安装 Neo4j](#an-zhuang-neo4j)
  * [别名](#bei-ming)
* [日常使用](#ri-chang-shi-yong)
  * [全局访问 Homestead](#quan-ju-fang-wen-homestead)
  * [通过 SSH 连接](#tong-guo-ssh-lian-jie)
  * [数据库连接](#shu-ju-ku-lian-jie)
  * [数据库备份](#shu-ju-ku-bei-fen)
  * [添加额外站点](#tian-jiaewai-de-zhan-dian)
  * [环境变量](#huan-jing-bian-liang)
  * [配置定时计划](#pei-zhi-ding-shi-ji-hua)
  * [配置 Mailhog](#pei-zhi-mailhog)
  * [配置 Minio](#pei-zhi-minio)
  * [端口](#duan-kou)
  * [共享你的环境](#gong-xiang-ni-de-huan-jing)
  * [多版本 PHP](#duo-ban-ben-php)
  * [Web 服务器](#web-fu-wu-qi)
  * [邮件](#mail)
* [网络接口](#wang-luo-jie-kou)
* [扩展 Homestead](#kuo-zhan-homestead)
* [更新 Homestead](#gen-xing-homestead)
* [提供特殊设置](#ti-gong-te-shu-she-zhi)
  * [VirtualBox](#virtualbox)

## 简介

Laravel 致力于让整个 PHP 开发体验变得愉快，包括你的本地开发环境。[Vagrant](https://www.vagrantup.com/) 提供一个简单，优雅的方式去管理和配置虚拟机。

Laravel Homestead 是一个官方的，预打包的 Vagrant 盒子，它为你提供了一个美妙的开发环境，而无需你安装 PHP，web 服务器和任何其它服务器软件在你的本地机器上。不在担心弄乱你的操作系统！Vagrant 盒子完全是一次性的。如果出现问题，你能销毁并在几分钟内重新创建盒子！

Homestead 运行在任何 Windows，Mac 或 Linux 系统上，并包含 Nginx web 服务器，PHP 7.3，PHP 7.2，PHP 7.1，MySQL，PostgreSQL，Redis，Memcached，Node 以及你需要开发出色的 Laravel 应用所需的所有其它好东西。

{% hint style="danger" %}

如果你使用 Windows，你需要开启硬件虚拟化（VT-x）。它通常是通过你的 BIOS 去开启。如果你在一个 UEFI 系统上使用 Hyper-V，为了访问 VT-x，你可能另外需要去禁用 Hyper-V。

{% endhint %}

### 包含软件

* Ubuntu 18.04
* Git
* PHP 7.3
* PHP 7.2
* PHP 7.1
* Nginx
* Apache（可选）
* MySQL
* MariaDB（可选）
* Sqlite3
* PostgreSQL
* Composer
* Node（包括 Yarn，Bower，Grunt 和 Gulp）
* Redis
* Memcached
* Beanstalkd
* Mailhog
* Neo4j（可选）
* MongoDB（可选）
* Elasticsearch（可选）
* ngrok
* wp-cli
* Zend Z-Ray
* Go
* Minio

## 安装与设置

### 起步

在运行 Homestead 环境之前，你必须安装 [VirtualBox](https://www.virtualbox.org/wiki/Downloads)，[VMWare](https://www.vmware.com/)，[Parallels](https://www.parallels.com/products/desktop/)，或者 [Hyper-V](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/quick-start/enable-hyper-v) 以及 [Vagrant](https://www.vagrantup.com/downloads.html)。所有这些软件包都为所有流行的操作系统提供了易于使用的可视化安装程序。

为了使用 VMWare 提供者，你将需要购买 VMWare / Workstation 和 [VMware Vagrant plug-in](https://www.vagrantup.com/vmware/)。尽管它不是免费的，VMWare 能提供开箱即用的更快的共享文件夹性能。

为了使用 Parallels 提供者，你将需要安装 [Parallels Vagrant plug-in](https://github.com/Parallels/vagrant-parallels)。它是免费的。

由于 [Vagrant 限制](https://www.vagrantup.com/docs/hyperv/limitations.html)，Hyper-V 提供者将忽略所有的网络设置。

#### 安装 Homestead Vagrant 盒子

一旦 VirtualBox / VMware 和 Vagrant 被安装，你应当在终端中使用如下命令将 `laravel/homestead` 盒子添加到你的 Vagrant 安装中。它将花费几分钟下载盒子，耗费的时间取决于你的网速：

```bash
vagrant box add laravel/homestead
```

如果这个命令失败，确保你安装的 Vagrant 是最新版。

#### 安装 Homestead

你可以安装 Homestead 通过克隆仓库。考虑克隆仓库到你的『home』目录下的一个 Homestead 文件夹下，这样 Homestead 盒子服务将作为你的所有 Laravel 项目的主机：

```bash
git clone https://github.com/laravel/homestead.git ~/Homestead
```

你应当检查 Homestead 的标记版本，因为 `master` 分支不总是稳定的。你能在 [GitHub Release Page](https://github.com/laravel/homestead/releases) 找到最新的稳定版：

```bash
cd ~/Homestead

// Clone the desired release...
git checkout v8.1.0
```

一旦你克隆克隆了 Homestead 仓库，从 Homestead 目录运行 `bash init.sh` 命令去创建 `Homestead.yaml` 配置文件。此 `Homestead.yaml` 文件将位于 Homestead 目录中：

```bash
// Mac / Linux...
bash init.sh

// Windows...
init.bat
```

### 配置 Homestead

#### 设置提供者

在你的 `Homestead.yaml` 文件中的 `provider` 键名表示哪个 `Vagrant` 提供者应当被使用：`virtualbox`，`vmware_fusion`，`vmware_workstation`，`parallels` 或者 `hyperv`。你可以设置你偏好的提供者：

```bash
provider: virtualbox
```

#### 配置共享文件夹

`Homestead.yaml` 文件中的 `folders` 属性列出了你希望与你的 `Homestead` 环境共享的所有文件夹。在这些文件夹下的文件发生改变时，他们将在你的本地机器和 Homestead 环境之间保持同步。你可以根据需要去配置许多共享文件夹。

```bash
folders:
    - map: ~/code
      to: /home/vagrant/code
```

如果你仅仅创建一些站点，这个通用的映射将正常工作。然而，随着站点数量的不断增加，你可能会遇到性能问题。在包含大量文件的低端机器或者项目中，这个问题可能非常明显且令人痛苦。如果你遇到这个问题，尝试将每个项目映射到其自己的 Vagrant 文件夹下：

```bash
folders:
    - map: ~/code/project1
      to: /home/vagrant/code/project1

    - map: ~/code/project2
      to: /home/vagrant/code/project2
```

要开启 [NFS](https://www.vagrantup.com/docs/synced-folders/nfs.html)，你仅仅需要增加一个简单的标志到你的同步文件配置中：

```bash
folders:
    - map: ~/code
      to: /home/vagrant/code
      type: "nfs"
```

{% hint style="danger" %}

当使用 NFS 时，你应当考虑安装 [vagrant-bindfs](https://github.com/winnfsd/vagrant-winnfsd) 插件。这个插件将为 Homestead 盒子中的文件和目录维护正确的 user / group 权限。

{% endhint %}

你也可以传递通过 `options` 键下列出它们在 Vagrant 的 [同步文件夹](https://www.vagrantup.com/docs/synced-folders/basic_usage.html) 中所支持的任何选项：

```bash
folders:
    - map: ~/code
      to: /home/vagrant/code
      type: "rsync"
      options:
          rsync__args: ["--verbose", "--archive", "--delete", "-zz"]
          rsync__exclude: ["node_modules"]
```

#### 配置 Nginx 站点

不熟悉 Nginx？没问题。`sites` 属性允许你在 Homestead 环境中轻松映射一个『域名』到一个文件夹。在 `Homestead.yaml` 文件中包含一个示例站点配置。接下来，你可以根据需要添加许多站点到你的 Homestead 环境。Homestead 能作为一个方便的，虚拟化的环境为你工作的每一个 Laravel 项目服务：

```bash
sites:
    - map: homestead.test
      to: /home/vagrant/code/my-project/public
```

如果你改变 `sites` 属性在配置 Homestead 盒子之后，你应当重新运行 `vagrant reload --provision` 去更新虚拟机中的 Nginx 配置。

#### Hosts文件

你必须在你机器的 `hosts` 文件中为 Nginx 站点添加『域名』。`hosts` 文件将把你的 Homestead 站点请求重定向到你的 Homestead 机器。在 Mac 和 Linux 中，这个文件位于 `/etc/hosts`。在 Windows 中，位于 `C:\Windows\System32\drivers\etc\hosts`。你添加到此文件中的行看起来像如下这样：

```bash
192.168.10.10  homestead.test
```

确保列出的 IP 地址是 `Homestead.yaml` 文件中设置的 IP 地址。一旦你添加域名到你的 `hosts` 文件，并启动 Vagrant 盒子后你能在你的 web 浏览器中访问它：

```bash
http://homestead.test
```

### 运行 Vagrant 盒子

一旦你根据你的喜好编辑 `Homestead.yaml` 后，从你的 Homestead 目录运行 `vagrant up` 命令。Vagrant 将启动虚拟机并自动配置你的共享文件夹和 Nginx 站点。

在销毁虚拟机，你可以使用 `vagrant destroy --force` 命令。

### 每个项目中安装

你可以为你所管理的每个项目配置配置一个 Homestead 实例，而不是在全局安装 Homestead 并在所有的项目中共享相同的 Homestead 盒子。如果你希望随项目一起发送一个 `Vagrantfile`，那么为每个项目安装 Homestead 可能是大有裨益的，允许其它人在项目中 `vagrant up` 进行工作。

要直接将 Homestead 安装到你的项目，需要使用 Composer：

```bash
composer require laravel/homestead --dev
```

一旦 Homestead 被安装，使用 `make` 命令去生成 `Vagrantfile` 和 `Homestead.yaml` 文件在你的项目根目录。`make` 命令将自动配置 `site` 和 `folders` 指令在 `Homestead.yaml` 文件中。

Mac / Linux：

```bash
php vendor/bin/homestead make
```

Windows：

```bash
vendor\\bin\\homestead make
```

接下来，在终端中运行 `vagrant up` 命令并在浏览器中输入 `http://homestead.test` 去访问你的项目。记住，你将仍然需要添加一个 `homestead.test` 或者你自定义的域名到 `/etc/hosts` 文件中。

### 安装 MariaDB

如果你偏好使用 MariaDB 而不是 MySQL，你可以在 `Homestead.yaml` 文件中添加 `mariadb` 选项。这个选项将移除 MySQL 并安装 MariaDB。MariaDB 服务作为 MySQL 的一个替代品，因此你仍然在你的应用程序数据库配置中使用 `mysql` 数据库驱动：

```bash
box: laravel/homestead
ip: "192.168.10.10"
memory: 2048
cpus: 4
provider: virtualbox
mariadb: true
```

### 安装 MongoDB

要安装 MongoDB 社区版，更新你的 `Homestead.yaml` 文件用如下的配置选项：

```bash
mongodb: true
```

默认的 MongoDB 安装将设置数据库用户名为 `homestead`，相应地密码为 `secret`。

### 安装 Elasticsearch

要安装 Elasticsearch，添加 `elasticsearch` 选项到你的 `Homestead.yaml` 文件并指定一个支持的版本，该版本可能是一个主版本或者一个精确的版本号（主版本号.次版本号.修复版本号）。默认安装将创建一个名为 `homestead` 的集群。你应当从不给 Elasticsearch 超过操作系统一半的内存，所以要确保你的 Homestead 机器至少有两倍的 Elasticsearch 分配的内存：

```bash
box: laravel/homestead
ip: "192.168.10.10"
memory: 4096
cpus: 4
provider: virtualbox
elasticsearch: 6
```

{% hint style="info" %}

查阅 [Elasticsearch 文件](https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html) 去学习如何自定义你的配置。

{% endhint %}

### 安装 Neo4j

[Neo4j](https://neo4j.com/) 是一个图形数据库管理系统。要安装 Neo4j 社区版，更新你的 `Homestead.yaml` 文件用如下的配置选项：

```bash
neo4j: true
```

默认的 Neo4j 安装将设置数据库的用户名为 `homestead`，相应的密码为 `secret`。要访问 Neo4j，用浏览器访问 `http://homestead.test:7474` 地址。你的端口 `7687`（Bolt），`7474`（HTTP），`7473`（HTTPS）是准备好服务来自 Neo4j 客户端的请求的。

### 别名

你可以通过在你的 Homestead 目录下修改 `aliases` 文件添加 Bash 别名到你的 Homestead 机器：

```bash
alias c='clear'
alias ..='cd ..'
```

随后你需要更新 `aliases` 文件，你应当使用 `vagrant reload --provision` 命令重新配置 Homestead 机器。这将确保在机器中你的新别名可用。

## 日常使用

### 全局访问 Homestead

有时你想在文件系统的任何地方 `vagrant up` 你的 Homestead 机器。你能这样做，在 Mac / Linux 系统上通过添加一个 Bash 函数到你的 Bash 简介中。在 Windows 中，你可以通过添加一个『批处理』文件到 `PATH` 来实现这个操作。这些脚本将允许你在你的系统的任何地方运行任何 Vagrant 命令，并自动将该命令指向你的 Homestead 安装目录：

#### Mac / Linux

```bash
function homestead() {
    ( cd ~/Homestead && vagrant $* )
}
```

确保函数中的 `~/Homestead` 路径调整到实际的 Homestead 安装的位置。一旦该函数被安装，你可以从你的系统的任何位置运行像 `homestead up` 或 `homestead ssh` 命令。

#### Windows

在你的机器的任何位置创建一个如下内容的 homestead.bat 批处理文件：

```bash
@echo off

set cwd=%cd%
set homesteadVagrant=C:\Homestead

cd /d %homesteadVagrant% && vagrant %*
cd /d %cwd%

set cwd=
set homesteadVagrant=
```

确保在脚本中的实例 `C:\Homestead` 路径调整为 Homestead 实际安装的位置。创建文件之后，添加文件位置到 `PATH` 中。然后，你可以在你的系统的任何地方运行像 `homestead up` 或者 `homestead ssh` 之类的命令。

### 通过 SSH 连接

你能从你的 Homestead 目录中通过在终端中发送 `vagrant ssh` 命令 SSH 到你的虚拟机中。

但是，由于你可能需要经常 SSH 到你的 Homestead 机器，考虑将上面的『函数』添加到主机以快速 SSH 到 Homestead 盒子中。

### 数据库连接

一个 `homestead` 数据库配置了开箱即用的 MySQL 和 PostgreSQL。为了更加方便，Laravel 的 `.env` 文件配置到框架以开箱即用此数据库。

要从你的主机的数据库客户端连接到你的 MySQL 或者 PostgreSQL 数据库，你应当连接 `127.0.0.1` 和端口为 `33060`（MySQL）或者 `54320`（PostgreSQL）。两个数据库的用户名和密码是 `homestead` / `secret`。

{% hint style="danger" %}

当从你的主机连接到数据库时，你应当仅使用这些非标准端口。由于 Laravel 在虚拟机中运行，你将在你的 Laravel 数据库配置文件中使用默认的 3306 和 5532 端口。

{% endhint %}

### 数据库备份

当你的 Vagrant 盒子销毁时，Homestead 能自动备份你的数据库。为了利用这个功能，你必须使用 Vagrant 2.1.0 或者更高的版本。或者，如果你使用一个旧版本的 Vagrant，你必须安装 `vagrant-triggers` 插件。为了开启自动数据库备份，添加如下的行到你的 `Homestead.yaml` 文件：

```bash
backup: true
```

一旦配置了，当 `vagrant destroy` 命令被执行时，Homestead 将导出你的数据库到 `mysql_backup` 和 `postgres_backup` 目录。如果你使用 [每个项目安装](https://laravel.com/docs/5.8/homestead#per-project-installation) 方法，则可以在克隆的 Homestead 或者在你的项目根目录中找到这些目录。

### 添加额外的站点

一旦你的 Homestead 环境已配置并运行，你可以为你的 Laravel 应用添加额外的 Nginx 站点。你可能希望在单个 Homestead 环境运行多个 Laravel 安装。要添加额外的站点，到 `Homestead.yaml` 文件中添加站点：

```bash
sites:
    - map: homestead.test
      to: /home/vagrant/code/my-project/public
    - map: another.test
      to: /home/vagrant/code/another/public
```

如果 Vagrant 不能自动管理你的『hosts』文件，你可能还需要添加新的站点到该文件中：

```bash
192.168.10.10  homestead.test
192.168.10.10  another.test
```

一旦这个站点被添加，从你的 Homestead 目录运行 `vagrant reload --provision` 命令。

#### 站点类型

Homestead 支持多种类型的站点，允许你轻松运行不是基于 Laravel 的项目。例如，我们可以使用 `symfony2` 站点类型轻松地添加一个 Symfony 应用到 Homestead：

```bash
sites:
    - map: symfony2.test
      to: /home/vagrant/code/my-symfony-project/web
      type: "symfony2"
```

可用的站点类型是：`apache`，`apigility`，`expressive`，`laravel`（默认），`proxy`，`silverstripe`，`statamic`，`symfony2`，`symfony4`，和 `zf`。

#### 站点参数

你可以通过 `params` 站点指令添加额外的 Nginx `fastcgi_param` 值到你的站点。例如，我们添加一个值为 `BAR` 的 `Foo` 参数：

```bash
sites:
    - map: homestead.test
      to: /home/vagrant/code/my-project/public
      params:
          - key: FOO
            value: BAR
```

### 环境变量

你能通过添加如下的值到 `Homestead.yaml` 文件来设置全局环境变量：

```bash
variables:
    - key: APP_ENV
      value: local
    - key: FOO
      value: bar
```

更新 `Homestead.yaml` 文件之后，确保通过运行 `vagrant reload --provision` 命令重新配置机器。这个将更新所有安装 的 PHP 版本的 PHP-FPM 配置并为 `vagrant` 用户更新环境。

### 配置定时计划

Laravel 提供了一种 [计划定时作业](https://laravel.com/docs/5.8/scheduling) 的方式，通过安排单个 `schedule:run` Artisan 命令去每分钟运行。`schedule:run` 命令将检查在你在 `App\Console\Kernel` 类中定义的计划来决定运行哪个作业。

如果你想为一个 Homestead 站点的 `schedule:run` 命令运行起来，在定义站点时，你可以设置 `schedule` 选项为 `true`：

```bash
sites:
    - map: homestead.test
      to: /home/vagrant/code/my-project/public
      schedule: true
```

站点的 Cron 作业将被定义在虚机的 `/etc/cron.d` 目录中。

### 配置 Mailhog

Mailhog 允许你轻松地捕获外发的电子邮件并进行检查它，而实际上没有将发送邮件给收件人。开始使用时，使用以下的邮件设置更新你的 `.env` 文件：

```bash
MAIL_DRIVER=smtp
MAIL_HOST=localhost
MAIL_PORT=1025
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null
```

一旦 Mailhog 被配置，你可以在 `http://localhost:8025` 上访问 Mailhog 控制面板。

### 配置 Minio

Minio 是一个开源的对象存储服务器，具有与 Amazon S3 兼容的 API。要安装 Minio，用以下的配置选项更新你的 `Homestead.yaml` 文件：

```bash
minio: true
```

默认情况下，Minio 在端口 `9600` 上是可用的。你可以访问 Minio 控制面板通过访问 `http://homestead:9600`。默认访问的键名是 `homestead`，同时默认的密钥是 `secretkey`。当访问 Minio 时，你应当总是使用 `us-east-1`。

为了使用 Minio，你将需要在你的 `config/filesystems.php` 配置文件中调整 S3 磁盘配置。你将需要添加 `use_path_style_endpoint` 选项到磁盘配置，并将 `url` 键更改为 `endpoint`：

```php
's3' => [
    'driver' => 's3',
    'key' => env('AWS_ACCESS_KEY_ID'),
    'secret' => env('AWS_SECRET_ACCESS_KEY'),
    'region' => env('AWS_DEFAULT_REGION'),
    'bucket' => env('AWS_BUCKET'),
    'endpoint' => env('AWS_URL'),
    'use_path_style_endpoint' => true
]
```

最后，确保你的 `.env` 文件有如下的选项：

```bash
AWS_ACCESS_KEY_ID=homestead
AWS_SECRET_ACCESS_KEY=secretkey
AWS_DEFAULT_REGION=us-east-1
AWS_URL=http://homestead:9600
```

为了配置存储桶，在 Homestead 配置文件中添加一个 `buckets` 指令：

```bash
buckets:
    - name: your-bucket
      policy: public
    - name: your-private-bucket
      policy: none
```

支持的 `policy` 值包括：`none`，`download`，`upload` 和 `public`。

### 端口

默认情况下，如下的端口将转发到你的 Homestead 环境：

* **SSH**：2222 -> 转发到 22
* **ngrok UI**：4040 -> 转发到 4040
* **HTTP**：8000 -> 转发到 80
* **HTTPS**：44300 -> 转发到 443
* **MySQL**：33060 -> 转发到 3306
* **PostgreSQL**：54320 -> 转发到 5432
* **MongoDB**：27017 -> 转发到 27017
* **Mailhog**：8025 -> 转发到 8025
* **Minio**：9600 -> 转发到 9600

#### 转发额外的端口

如果你愿意，你可以转发额外的端口到 Vagrant 盒子，同时也指定他们的协议：

```bash
ports:
    - send: 50000
      to: 5000
    - send: 7777
      to: 777
      protocol: udp
```

### 共享你的环境

有时你希望共享你当前工作的内容跟同事或客户。Vagrant 有内置的方法通过 `vagrant share` 去支持这个；然而，如果在 `Homestead.yaml` 文件中有多个站点配置，这个将不会工作。

为了解决这个问题，Homestead 包含它自己的 `share` 命令。开始时，SSH 通过 `vagrant ssh` 进入你的 Homestead 机器并运行 `share homestead.test`。这个将从你的 `Homestead.yaml` 配置文件中分享 `homestead.test` 站点。当然，你可以将任何其它配置的站点替换为 `homestead.test`：

```bash
share homestead.test
```

运行此命令之后，你将看到一个 Ngrok 屏幕出现，其中包含了活动日志和共享站点的可访问的 URLs。如果你想指定一个自定义的区域，子域，或者其它 Ngrok 运行时选项，你可以添加他们到你的 `share` 命令：

```bash
share homestead.test -region=eu -subdomain=laravel
```

{% hint style="danger" %}

记住，Vagrant 本质上是不安全的，并且在运行 `share` 命令时将虚拟机显露到互联网。

{% endhint %}

### 多版本 PHP

Homestead 6 在同一个虚拟机上引入了对多个 PHP 版本的支持。你可以在你的 `Homestead.yaml` 文件中指定要用于给定站点的 PHP 版本。可用的 PHP 版本是：『7.1』，『7.2』和『7.3』（默认）：

```bash
sites:
    - map: homestead.test
      to: /home/vagrant/code/my-project/public
      php: "7.1"
```

另外，你可以通过 CLI 使用任何支持的 PHP 版本：

```bash
php7.1 artisan list
php7.2 artisan list
php7.3 artisan list
```

### Web 服务器

默认情况下，Homestead 使用 Nginx web 服务器。然而，如果 `apache` 是指定的一个站点类型，它能安装 Apache。同时两个 web 服务器在同时被安装，他们不能同时运行。`flip` 脚本命令可用来轻松处理 web 服务器之间的切换。`flip` 命令自动确定正在运行的 web 服务器，将其关闭。然后启动其它服务器。要使用这个命令，SSH 进入到你的 Homestead 机器并在终端运行命令：

```bash
flip
```

### Mail

Homestead 包括 Postfix 邮件传输代理，默认监听 `1025` 端口。因此，你可以指示你的应用程序在 `localhost` 端口 `1025` 上使用 `smtp` 邮件驱动。所有发送的邮件将由 Postfix 处理并由 Mailhog 捕获。要查看已发送的邮件，在 web 浏览器中打开 `http://localhost:8025`。

## 网络接口

`Homestead.yaml` 中的 `networks` 属性为 Homestead 环境配置网络接口。你可以根据需要配置多个接口：

```bash
networks:
    - type: "private_network"
      ip: "192.168.10.20"
```

要开启一个 [桥接](https://www.vagrantup.com/docs/networking/public_network.html) 接口，配置 `bridge` 设置并改变 `public_network` 网络类型：

```bash
networks:
    - type: "public_network"
      ip: "192.168.10.20"
      bridge: "en1: Wi-Fi (AirPort)"
```

要开启 [DHCP](https://www.vagrantup.com/docs/networking/public_network.html)，仅从你的配置中移除 `ip` 选项：

```bash
networks:
    - type: "public_network"
      bridge: "en1: Wi-Fi (AirPort)"
```

## 扩展 Homestead

你能在 Homestead 根目录下使用 `after.sh` 脚本扩展 Homestead。在这个文件中，你可以添加正确配置和自定义虚拟机所需的任何 shell 命令。

当自定义 Homestead 时，Ubuntu 可能会询问你是否要保留程序包的原始配置或者用一个新的配置文件覆盖它。为了避免这种情况，你应该在安装软件包时使用以下的命令，以避免覆盖之前由 Homestead 编写的任何配置：

```bash
sudo apt-get -y \
    -o Dpkg::Options::="--force-confdef" \
    -o Dpkg::Options::="--force-confold" \
    install your-package
```

## 更新 Homestead

你能通过一些简单的步骤更新 Homestead。首先，你应当使用 `vagrant box update` 命令更新 Vagrant 盒子：

```bash
vagrant box update
```

接下来，你需要去更新 Homestead 源代码。如果你克隆过仓库，你能在最初克隆仓库的位置运行如下的命令：

```bash
git fetch

git checkout v8.0.1
```

这些命令从 GitHub 仓库拉取最新的 Homestead 代码，获取最新的标记，然后检出最新的标记版本。你能在 [GitHub 版本页面](https://github.com/laravel/homestead/releases) 找到最新稳定的版本。

如果你通过你的项目的 `composer.json` 安装 Homestead，你应当确保你的 `composer.json` 包含 `"laravel/homestead": "^8"` 并更新你的依赖项：

```bash
composer update
```

最后，你将需要销毁和重新生成你的 Homestead 盒子去利用最新的 Vagrant 安装。为了实现这个，在你的 Homestead 目录运行如下的命令：

```bash
vagrant destroy

vagrant up
```

## 提供特殊设置

### VirtualBox

#### natdnshostresolver

默认情况下，Homestead 将 `natdnshostresolver` 配置设置为 `on`。这允许 Homestead 去使用你的主机操作系统的 DNS 设置。如果你不想覆盖这个行为，添加如下的行到你的 `Homestead.yaml` 文件：

```bash
provider: virtualbox
natdnshostresolver: off
```

#### Windows 上的符号链接

如果符号链接在你的 Windows 机器上无法正常工作，你可能需要添加以下的块到 `Vagrantfile` 文件：

```bash
config.vm.provider "virtualbox" do |v|
    v.customize ["setextradata", :id, "VBoxInternal2/SharedFoldersEnableSymlinksCreate/v-root", "1"]
end
```
