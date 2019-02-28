# Homestead

* [简介](#jian-jie)
* [安装与设置](#an-zhuang-yu-she-zhi)
* [日常使用](#ri-chang-shi-yong)
* [网络接口](#wang-luo-jie-kou)
* [扩展 Homestead](#kuo-zhan-homestead)
* [更新 Homestead](#gen-xing-homestead)
* [提供特殊设置](#ti-gong-te-shu-she-zhi)

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

#### 运行 Vagrant 盒子

一旦你根据你的喜好编辑 `Homestead.yaml` 后，从你的 Homestead 目录运行 `vagrant up` 命令。Vagrant 将启动虚拟机并自动配置你的共享文件夹和 Nginx 站点。

在销毁虚拟机，你可以使用 `vagrant destroy --force` 命令。

#### 每个项目中安装

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

#### 安装 MariaDB

如果你偏好使用 MariaDB 而不是 MySQL，你可以在 `Homestead.yaml` 文件中添加 `mariadb` 选项。这个选项将移除 MySQL 并安装 MariaDB。MariaDB 服务作为 MySQL 的一个替代品，因此你仍然在你的应用程序数据库配置中使用 `mysql` 数据库驱动：

```bash
box: laravel/homestead
ip: "192.168.10.10"
memory: 2048
cpus: 4
provider: virtualbox
mariadb: true
```

#### 安装 MongoDB

要安装 MongoDB 社区版，更新你的 `Homestead.yaml` 文件用如下的配置选项：

```bash
mongodb: true
```

默认的 MongoDB 安装将设置数据库用户名为 `homestead`，相应地密码为 `secret`。

#### 安装 Elasticsearch

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

#### 安装 Neo4j

[Neo4j](https://neo4j.com/) 是一个图形数据库管理系统。要安装 Neo4j 社区版，更新你的 `Homestead.yaml` 文件用如下的配置选项：

```bash
neo4j: true
```

默认的 Neo4j 安装将设置数据库的用户名为 `homestead`，相应的密码为 `secret`。要访问 Neo4j，用浏览器访问 `http://homestead.test:7474` 地址。你的端口 `7687`（Bolt），`7474`（HTTP），`7473`（HTTPS）是准备好服务来自 Neo4j 客户端的请求的。

#### 别名

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

### 环境变量

### 配置定时计划

### 配置 Mailhog

### 配置 Minio

## 网络接口

## 扩展 Homestead

## 更新 Homestead

## 提供特殊设置
