# 文件存储

## 简介

Laravel 提供了一个强大的文件系统抽象，这要感谢 Frank de Jonge 提供的非常棒的 [Flysystem](https://github.com/thephpleague/flysystem) PHP 包。Laravel Flysystem 集成为使用本地文件系统、Amazon S3 和 Rackspace 云存储提供了简单易用的驱动程序。更好的是，在这些存储选项之间切换非常简单，因为每个系统的 API 都是相同的。

## 配置

文件系统配置文件位于 `config/filesystems.php`。在此文件中，你可以配置所有『磁盘』。 每个磁盘代表特定的存储驱动程序和存储位置。配置文件中包含每个支持的驱动程序的示例配置。因此，请修改配置以反映你的存储首选项和凭证。

你可以按你的希望配置多个磁盘，甚至可以有多个使用相同驱动程序的磁盘。

### 公共磁盘

`public` 磁盘为可公开访问的文件做打算。默认情况下，`public` 磁盘使用 `local` 驱动程序并将这些文件存储在 `storage/app/public` 中。要使它们可以从 Web 访问，你应该创建一个从 `public/storage` 到 `storage/app/public` 的符号链接。此约定将使你可公开访问的文件保存在一个目录中，当使用 [Envoyer](https://envoyer.io/) 等零停机时间部署系统时，可以在部署中轻松共享这些文件。

要创建符号链接，你可以使用 `storage:link` Artisan 命令：

```bash
php artisan storage:link
```

一旦文件被存储并创建了符号链接，你就可以使用 `asset` 助手创建到文件的 URL 了。

```php
echo asset('storage/file.txt');
```

### 本地驱动

使用 `local` 驱动程序时，所有文件操作都与配置文件中定义的 `root` 目录有关。默认情况下，此值设置为 `storage/app` 目录。因此，以下方法将文件存储在 `storage/app/file.txt` 中：

```php
Storage::disk('local')->put('file.txt', 'Contents');
```

### 驱动先决条件

#### Composer 包

使用 SFTP、S3 或者 Rackspace 驱动之前，你将需要通过 Composer 去安装合适的包：

* SFTP: `league/flysystem-sftp ~1.0`
* Amazon S3: `league/flysystem-aws-s3-v3 ~1.0`
* Rackspace: `league/flysystem-rackspace ~1.0`

要提高性能必须使用缓存的适配器。你将需要一个额外的软件包：

* CachedAdapter: `league/flysystem-cached-adapter ~1.0`

#### S3 驱动配置

S3 驱动程序配置信息位于 `config/filesystems.php` 配置文件中。此文件包含 S3 驱动程序的示例配置数组。你可以使用自己的 S3 配置和凭证自由修改此数组。为方便起见，这些环境变量与 AWS CLI 使用的命名约定相匹配。

#### FTP 驱动配置

Laravel 的 Flysystem 集成与 FTP 一起工作的非常好；但是，框架的默认 `filesystems.php` 配置文件中没有包含示例配置。如果你需要配置 FTP 文件系统，你可以使用下面的示例配置：

```php
'ftp' => [
    'driver'   => 'ftp',
    'host'     => 'ftp.example.com',
    'username' => 'your-username',
    'password' => 'your-password',

    // 可选的 FTP 设置...
    // 'port'     => 21,
    // 'root'     => '',
    // 'passive'  => true,
    // 'ssl'      => true,
    // 'timeout'  => 30,
],
```

#### SFTP 驱动配置

Laravel 的 Flysystem 集成与 SFTP 一起工作的非常好；但是，框架的默认 `filesystems.php` 配置文件中没有包含示例配置。如果需要配置 SFTP 文件系统，你可以使用下面的示例配置：

```php
'sftp' => [
    'driver' => 'sftp',
    'host' => 'example.com',
    'username' => 'your-username',
    'password' => 'your-password',

    // 基于 SSH 密钥的认证设置...
    // 'privateKey' => '/path/to/privateKey',
    // 'password' => 'encryption-password',

    // 可选的 SFTP 设置...
    // 'port' => 22,
    // 'root' => '',
    // 'timeout' => 30,
],
```

#### Packspace 驱动配置

Laravel 的 Flysystem 集成与 Rackspace 一起工作的非常好；但是，框架的默认 `filesystems.php` 配置文件中没有包含示例配置。如果需要配置 Rackspace 文件系统，你可以使用下面的示例配置：

```php
'rackspace' => [
    'driver'    => 'rackspace',
    'username'  => 'your-username',
    'key'       => 'your-key',
    'container' => 'your-container',
    'endpoint'  => 'https://identity.api.rackspacecloud.com/v2.0/',
    'region'    => 'IAD',
    'url_type'  => 'publicURL',
],
```

### 缓存

要为给定磁盘启用缓存，你可以去磁盘的配置选项中添加 `cache` 指令。`cache` 选项应该是包含 `disk` 名称、以秒为单位的 `expire` 时间和缓存 `prefix` 的缓存选项数组：

```php
's3' => [
    'driver' => 's3',

    // 其它磁盘选项...

    'cache' => [
        'store' => 'memcached',
        'expire' => 600,
        'prefix' => 'cache-prefix',
    ],
],
```

## 获取磁盘实例

`Storage` 外观可用于与你的任何已配置的磁盘进行交互。例如，你可以使用外观上的 `put` 方法在默认磁盘上存储一个头像。如果你在没有首先调用 `disk` 方法的情况下调用 `Storage` 外观上的方法，那么方法调用将自动传递到默认磁盘：

```php
use Illuminate\Support\Facades\Storage;

Storage::put('avatars/1', $fileContents);
```

如果你的应用程序与多个磁盘交互，你可以使用 `Storage` 外观上的 `disk` 方法处理特定磁盘上的文件：

```php
Storage::disk('s3')->put('avatars/1', $fileContents);
```

## 检索文件

## 存储文件

## 删除文件

## 目录

## 自定义的文件系统
