# HTTP 测试

## 简介

Laravel 提供了一个非常流畅的 API，用于向你的应用程序发出 HTTP 请求并检查输出。例如，看看下面定义的测试：

```php
<?php

namespace Tests\Feature;

use Tests\TestCase;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Illuminate\Foundation\Testing\WithoutMiddleware;

class ExampleTest extends TestCase
{
    /**
     * 一个基本的测试实例。
     *
     * @return void
     */
    public function testBasicTest()
    {
        $response = $this->get('/');

        $response->assertStatus(200);
    }
}
```

***

`get` 方法向应用程序发出 `GET` 请求，而 `assertStatus` 方法断言返回的响应应该具有给定的 HTTP 状态代码。除了这个简单的断言之外，Laravel 还包含用于检查响应头、内容、JSON 结构等的各种断言。

### 自定义请求头

你可以使用 `withHeaders` 方法在将请求的标头发送到应用程序之前对其进行自定义。这允许你添加你想要的任何自定义标头：

```php
<?php

class ExampleTest extends TestCase
{
    /**
     * 一个基本的方法测试实例。
     *
     * @return void
     */
    public function testBasicExample()
    {
        $response = $this->withHeaders([
            'X-Header' => 'Value',
        ])->json('POST', '/user', ['name' => 'Sally']);

        $response
            ->assertStatus(201)
            ->assertJson([
                'created' => true,
            ]);
    }
}
```

***

{% hint style="info" %}

运行测试时会自动禁用 CSRF 中间件。

{% endhint %}

### 调试响应

在对你的应用程序发出测试请求后，可以使用 `dump` 和 `dumpHeaders` 方法来检查和调试响应内容：

```php
<?php

namespace Tests\Feature;

use Tests\TestCase;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Illuminate\Foundation\Testing\WithoutMiddleware;

class ExampleTest extends TestCase
{
    /**
     * 一个基本的测试实例。
     *
     * @return void
     */
    public function testBasicTest()
    {
        $response = $this->get('/');

        $response->dumpHeaders();

        $response->dump();
    }
}
```

***

## 会话 / 认证

Laravel 提供了几个助手程序，用于在 HTTP 测试期间使用会话。首先，你可以使用 `withSession` 方法将会话数据设置为给定数组。在向你的应用程序发出请求之前，这对于使用数据加载会话非常有用：

```php
<?php

class ExampleTest extends TestCase
{
    public function testApplication()
    {
        $response = $this->withSession(['foo' => 'bar'])
                         ->get('/');
    }
}
```

***

会话的一个常见用途是维护认证用户的状态。`actingAs` 辅助方法提供了一种简单的方法来将给定用户作为当前用户进行认证。例如，我们可以使用 [模型工厂](https://laravel.com/docs/5.8/database-testing#writing-factories) 来生成和认证用户：

```php
<?php

use App\User;

class ExampleTest extends TestCase
{
    public function testApplication()
    {
        $user = factory(User::class)->create();

        $response = $this->actingAs($user)
                         ->withSession(['foo' => 'bar'])
                         ->get('/');
    }
}
```

***

你还可以通过将保护名称作为第二个参数传递给 `actsAs` 方法来指定应该使用哪个保护来认证给定用户：

```php
$this->actingAs($user, 'api')
```

***

## 测试 JSON API

Laravel 还提供了几个助手来测试 JSON API 及其响应。例如：`json`、`get`、`post`、`put`、`patch`、`delete` 和 `option` 方法可用于发出具有各种 HTTP 动词的请求。你也可以轻松地将数据和标头传递给这些方法。首先，让我们编写一个测试对 `/user` 发出 `POST` 请求，并断言返回了预期的数据：

```php
<?php

class ExampleTest extends TestCase
{
    /**
     * 一个基本的功能测试示例。
     *
     * @return void
     */
    public function testBasicExample()
    {
        $response = $this->json('POST', '/user', ['name' => 'Sally']);

        $response
            ->assertStatus(201)
            ->assertJson([
                'created' => true,
            ]);
    }
}
```

***

{% hint style="info" %}

`assertJson` 方法将响应转换为数组，并使用 `PHPUnit::assertArraySubset` 来验证给定的数组是否存在于应用程序返回的 JSON 响应中。因此，如果 JSON 响应中还有其他属性，那么只要存在给定的片段，这个测试就会通过。

{% endhint %}

### 验证精确的 JSON 匹配

如果你想验证给定的数组是通过应用程序返回的 JSON 的一个 **精确**匹配，你应使用 `assertExactJson` 方法：

```php
<?php

class ExampleTest extends TestCase
{
    /**
     * 一个基本的功能测试示例。
     *
     * @return void
     */
    public function testBasicExample()
    {
        $response = $this->json('POST', '/user', ['name' => 'Sally']);

        $response
            ->assertStatus(201)
            ->assertExactJson([
                'created' => true,
            ]);
    }
}
```

***

## 测试文件上传

`Illuminate\Http\UploadedFile` 提供了一个 `fake` 方法，此方法可用于生成虚拟文件或图像以进行测试。这与 `Storage` 外观的 `fake` 方法相结合大大简化了文件上传的测试。例如，你可以结合使用这两个功能轻松测试头像上传表单：

```php
<?php

namespace Tests\Feature;

use Tests\TestCase;
use Illuminate\Http\UploadedFile;
use Illuminate\Support\Facades\Storage;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Illuminate\Foundation\Testing\WithoutMiddleware;

class ExampleTest extends TestCase
{
    public function testAvatarUpload()
    {
        Storage::fake('avatars');

        $file = UploadedFile::fake()->image('avatar.jpg');

        $response = $this->json('POST', '/avatar', [
            'avatar' => $file,
        ]);

        // 断言文件已存储...
        Storage::disk('avatars')->assertExists($file->hashName());

        // 断言文件不存在...
        Storage::disk('avatars')->assertMissing('missing.jpg');
    }
}
```

***

**假文件定制**

使用 `fake` 方法创建文件时，你可以指定图像的宽度、高度和大小，以便更好地测试验证规则：

```php
UploadedFile::fake()->image('avatar.jpg', $width, $height)->size(100);
```

***

除了创建图像，你还可以使用 `create` 方法创建任何其他类型的文件：

```php
UploadedFile::fake()->create('document.pdf', $sizeInKilobytes);
```

***

## 可用的断言

### 响应断言

Laravel 为你的 [PHPUnit](https://phpunit.de) 测试提供了各种自定义断言方法。可以从 `json`、`get`、`post`、`put` 和 `delete` 测试方法返回的响应中访问这些断言：

|                                                                                      |                                                                                  |
| ------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------- |
| [assertCookie](https://laravel.com/docs/5.8/http-tests#assert-cookie)                | [assertPlainCookie](https://laravel.com/docs/5.8/http-tests#assert-plain-cookie) |
| [assertCookieExpired](https://laravel.com/docs/5.8/http-tests#assert-cookie-expired) | [assertRedirect](https://laravel.com/docs/5.8/http-tests#assert-redirect)        |
