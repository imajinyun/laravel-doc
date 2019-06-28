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

`get` 方法可用于检索文件的内容。该方法将返回文件的原始字符串内容。记住，所有文件路径都应当指定为相对于为磁盘配置的『根』位置：

```php
$contents = Storage::get('file.jpg');
```

`exists` 方法可以用于确定一个文件是否在磁盘上存在：

```php
$exists = Storage::disk('s3')->exists('file.jpg');
```

### 下载文件

`download` 方法可用于生成一个响应，该响应强制用户的浏览器按给定路径下载文件。`download` 方法接受文件名作为其第二个参数，该参数将确定下载文件的用户所看到的文件名。最后，可以将 HTTP 头数组作为第三个参数传递给此方法：

```php
return Storage::download('file.jpg');

return Storage::download('file.jpg', $name, $headers);
```

### 文件 URLs

你可以使用 `url` 方法获取给定文件的 URL。如果你使用的是 `local` 驱动程序，则通常只会在给定路径中预先添加 `/storage` 并返回该文件的相对 URL。如果你使用的是 `s3` 或 `rackspace` 驱动程序，则将返回完全限定的远程 URL：

```php
use Illuminate\Support\Facades\Storage;

$url = Storage::url('file.jpg');
```

{% hint style="danger" %}

记住，如果你使用 `local` 驱动，所有应当公开访问应放置在 `storage/app/public` 目录中。此外，你应该在 `public/storage` 处创建一个指向 `storage/app/public` 目录的 [符号链接](https://laravel.com/docs/5.8/filesystem#the-public-disk)。

{% endhint %}

#### 临时 URLs

对于使用 `s3` 或 `rackspace` 驱动程序存储的文件，你可以使用 `temporaryUrl` 方法为给定文件创建临时 URL。此方法接受指定 URL 何时到期的一个路径和 `DateTime` 实例：

```php
$url = Storage::temporaryUrl(
    'file.jpg', now()->addMinutes(5)
);
```

如果你需要去指定额外的 [S3 请求参数](https://docs.aws.amazon.com/AmazonS3/latest/API/RESTObjectGET.html#RESTObjectGET-requests)，你可以在 `temporaryUrl` 方法上传递请求数组作为其第三个参数：

```php
$url = Storage::temporaryUrl(
    'file.jpg',
    now()->addMinutes(5),
    ['ResponseContentType' => 'application/octet-stream'],
);
```

#### 本地 URL 主机定制

如果你希望为使用 `local` 驱动程序存储在磁盘上的文件预先定义主机，你可以向磁盘的配置数组添加 `url` 选项：

```php
'public' => [
    'driver' => 'local',
    'root' => storage_path('app/public'),
    'url' => env('APP_URL').'/storage',
    'visibility' => 'public',
],
```

### 文件元数据

除了读取和写入文件外，Laravel 还可以提供关于文件本身的信息。例如，`size` 方法可用于以获取文件的字节大小：

```php
use Illuminate\Support\Facades\Storage;

$size = Storage::size('file.jpg');
```

`lastModified` 方法返回文件最后一次修改的 UNIX 时间戳：

```php
$time = Storage::lastModified('file.jpg');
```

## 存储文件

`put` 方法可用于将原始文件内容存储在磁盘上。你还可以将 PHP `resource` 传递给 `put` 方法，该方法将使用 Flysystem 的底层流支持。在处理大文件时，强烈建议使用流：

```php
use Illuminate\Support\Facades\Storage;

Storage::put('file.jpg', $contents);

Storage::put('file.jpg', $resource);
```

**自动流**

如果你希望 Laravel 自动管理将一个给定文件流到存储位置的流，可以使用 `putFile` 或 `putFileAs` 方法。此方法接受一个 `Illuminate\Http\File` 或 `Illuminate\Http\UploadedFile` 实例，并自动将文件流到所需位置：

```php
use Illuminate\Http\File;
use Illuminate\Support\Facades\Storage;

// 自动生成文件名的一个唯一 ID...
Storage::putFile('photos', new File('/path/to/photo'));

// 手动指定一个文件名...
Storage::putFileAs('photos', new File('/path/to/photo'), 'photo.jpg');
```

关于 `putFile` 方法，有几件重要的事情需要注意。注意，我们只指定了目录名，而没有指定文件名。默认情况下，`putFile` 方法将生成一个惟一的 ID 作为文件名。文件的扩展名将通过检查文件的 MIME 类型来确定。文件的路径将由 `putFile` 方法返回，这样你就可以将路径（包括生成的文件名）存储在数据库中。

`putFile` 和 `putFileAs` 方法还接受一个参数来指定存储文件的『可见性』。如果你将文件存储在诸如 S3 这样的云磁盘上，并且希望该文件可以公开访问，那么这一点特别有用：

```php
Storage::putFile('photos', new File('/path/to/photo'), 'public');
```

**添加 & 追加到文件**

`prepend` 和 `append` 方法允许你在文件的开头或结尾写入内容：

```php
Storage::prepend('file.log', 'Prepended Text');

Storage::append('file.log', 'Appended Text');
```

**复制 & 移动文件**

`copy` 方法可用于将存在的文件复制到磁盘上的新位置，而 `move` 方法可用于重命名或将存在的文件移动到一个新的位置：

```php
Storage::copy('old/file.jpg', 'new/file.jpg');

Storage::move('old/file.jpg', 'new/file.jpg');
```

### 文件上传

在 Web 应用程序中，用于存储文件的最常见用例之一是存储用户上传的文件，例如个人资料图片，照片和文档。Laravel 使用 `store` 方法在上传的文件实例上很容易存储上传的文件。使用你希望存储上传文件的路径上调用 `store` 方法：

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use App\Http\Controllers\Controller;

class UserAvatarController extends Controller
{
    /**
     * 为用户更新头像。
     *
     * @param  Request  $request
     * @return Response
     */
    public function update(Request $request)
    {
        $path = $request->file('avatar')->store('avatars');

        return $path;
    }
}
```

关于这个例子，有几件重要的事情需要注意。注意，我们只指定了目录名，而没有指定文件名。默认情况下，`store` 方法将生成一个惟一 ID 作为文件名。文件的扩展名将通过检查文件的 MIME 类型来确定。`store` 方法将返回文件的路径，以便你可以将路径（包括生成的文件名）存储在数据库中。

你可以在 `Storage` 外观上调用 `putFile` 方法去执行与上面示例相同的文件操作：

```php
$path = Storage::putFile('avatars', $request->file('avatar'));
```

#### 指定一个文件名称

如果你不希望将文件名自动分配给你的存储文件，你可以使用 `storeAs` 方法，它接收路径、文件名和（可选的）磁盘作为其参数：

```php
$path = $request->file('avatar')->storeAs(
    'avatars', $request->user()->id
);
```

你还可以使用在 `Storage` 外观上的 `putFileAs` 方法，它将执行与上面示例相同的文件操作：

```php
$path = Storage::putFileAs(
    'avatars', $request->file('avatar'), $request->user()->id
);
```

#### 指定一个磁盘

默认情况下，此方法将使用你的默认磁盘。如果你希望指定另一个磁盘，将磁盘名称作为第二个参数传递给 `store` 方法：

```php
$path = $request->file('avatar')->store(
    'avatars/'.$request->user()->id, 's3'
);
```

### 文件可见性

在 Laravel 的 Flysystem 集成中，『可见性』是跨多个平台的文件权限的抽象。文件可以声明为 `public` 或 `private`。当文件被声明为 `public` 时，你指示的该文件通常应该被其他人访问。例如，使用 S3 驱动程序时，你可以检索 `public` 文件的 URL。

你可以通过 `put` 方法设置文件时设置可见性：

```php
use Illuminate\Support\Facades\Storage;

Storage::put('file.jpg', $contents, 'public');
```

如果文件已经存储，则可以通过 `getVisibility` 和 `setVisibility` 方法检索和设置文件的可见性：

```php
$visibility = Storage::getVisibility('file.jpg');

Storage::setVisibility('file.jpg', 'public')
```

## 删除文件

`delete` 方法接受要从磁盘中删除的单个文件名或文件数组：

```php
use Illuminate\Support\Facades\Storage;

Storage::delete('file.jpg');

Storage::delete(['file.jpg', 'file2.jpg']);
```

如果需要，你可以指定文件应该从哪个磁盘删除：

```php
use Illuminate\Support\Facades\Storage;

Storage::disk('s3')->delete('folder_path/file_name.jpg');
```

## 目录

### 获取目录中的所有文件

`files` 方法返回给定目录中所有文件的数组。如果你希望检索给定目录（包括所有子目录）中的所有文件列表，可以使用 `allFiles` 方法：

```php
use Illuminate\Support\Facades\Storage;

$files = Storage::files($directory);

$files = Storage::allFiles($directory);
```

### 获取一个目录中的所有目录

`directories` 方法返回给定目录中所有目录的数组。此外，可以使用 `allDirectory` 方法获得给定目录及其所有子目录中的所有目录的列表：

```php
$directories = Storage::directories($directory);

// 递归...
$directories = Storage::allDirectories($directory);
```

### 创建一个目录

`makeDirectory` 方法将创建给定的目录，包括任何需要的子目录：

```php
Storage::makeDirectory($directory);
```

### 删除一个目录

最后，可以使用 `deleteDirectory` 方法删除一个目录及其所有文件：

```php
Storage::deleteDirectory($directory);
```

## 自定义的文件系统

Laravel 的 Flysystem 集成为开箱即用的几个『驱动』提供驱动程序；但是，Flysystem 不仅限于这些，并且具有适用于许多其他存储系统的适配器。如果要在 Laravel 应用程序中使用这些附加适配器之一，则可以创建自定义驱动程序。

为了设置自定义文件系统，你需要一个 Flysystem 适配器。让我们将社区维护的 Dropbox 适配器添加到我们的项目中：

```bash
composer require spatie/flysystem-dropbox
```
