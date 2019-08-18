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

|                                                                                                                    |                                                                                                            |
| ------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------- |
| [assertCookie](https://laravel.com/docs/5.8/http-tests#assert-cookie)                                              | [assertPlainCookie](https://laravel.com/docs/5.8/http-tests#assert-plain-cookie)                           |
| [assertCookieExpired](https://laravel.com/docs/5.8/http-tests#assert-cookie-expired)                               | [assertRedirect](https://laravel.com/docs/5.8/http-tests#assert-redirect)                                  |
| [assertCookieNotExpired](https://laravel.com/docs/5.8/http-tests#assert-cookie-not-expired)                        | [assertSee](https://laravel.com/docs/5.8/http-tests#assert-see)                                            |
| [assertCookieMissing](https://laravel.com/docs/5.8/http-tests#assert-cookie-missing)                               | [assertSeeInOrder](https://laravel.com/docs/5.8/http-tests#assert-see-in-order)                            |
| [assertDontSee](https://laravel.com/docs/5.8/http-tests#assert-dont-see)                                           | [assertSeeText](https://laravel.com/docs/5.8/http-tests#assert-see-text)                                   |
| [assertDontSeeText](https://laravel.com/docs/5.8/http-tests#assert-dont-see-text)                                  | [assertSeeTextInOrder](https://laravel.com/docs/5.8/http-tests#assert-see-text-in-order)                   |
| [assertExactJson](https://laravel.com/docs/5.8/http-tests#assert-exact-json)                                       | [assertSessionHas](https://laravel.com/docs/5.8/http-tests#assert-session-has)                             |
| [assertForbidden](https://laravel.com/docs/5.8/http-tests#assert-forbidden)                                        | [assertSessionHasInput](https://laravel.com/docs/5.8/http-tests#assert-session-has-input)                  |
| [assertHeader](https://laravel.com/docs/5.8/http-tests#assert-header)                                              | [assertSessionHasAll](https://laravel.com/docs/5.8/http-tests#assert-session-has-all)                      |
| [assertHeaderMissing](https://laravel.com/docs/5.8/http-tests#assert-header-missing)                               | [assertSessionHasErrors](https://laravel.com/docs/5.8/http-tests#assert-session-has-errors)                |
| [assertJson](https://laravel.com/docs/5.8/http-tests#assert-json)                                                  | [assertSessionHasErrorsIn](https://laravel.com/docs/5.8/http-tests#assert-session-has-errors-in)           |
| [assertJsonCount](https://laravel.com/docs/5.8/http-tests#assert-json-count)                                       | [assertSessionHasNoErrors](https://laravel.com/docs/5.8/http-tests#assert-session-has-no-errors)           |
| [assertJsonFragment](https://laravel.com/docs/5.8/http-tests#assert-json-fragment)                                 | [assertSessionDoesntHaveErrors](https://laravel.com/docs/5.8/http-tests#assert-session-doesnt-have-errors) |
| [assertJsonMissing](https://laravel.com/docs/5.8/http-tests#assert-json-missing)                                   | [assertSessionMissing](https://laravel.com/docs/5.8/http-tests#assert-session-missing)                     |
| [assertJsonMissingExact](https://laravel.com/docs/5.8/http-tests#assert-json-missing-exact)                        | [assertStatus](https://laravel.com/docs/5.8/http-tests#assert-status)                                      |
| [assertJsonMissingValidationErrors](https://laravel.com/docs/5.8/http-tests#assert-json-missing-validation-errors) | [assertSuccessful](https://laravel.com/docs/5.8/http-tests#assert-successful)                              |
| [assertJsonStructure](https://laravel.com/docs/5.8/http-tests#assert-json-structure)                               | [assertUnauthorized](https://laravel.com/docs/5.8/http-tests#assert-unauthorized)                          |
| [assertJsonValidationErrors](https://laravel.com/docs/5.8/http-tests#assert-json-validation-errors)                | [assertViewHas](https://laravel.com/docs/5.8/http-tests#assert-view-has)                                   |
| [assertLocation](https://laravel.com/docs/5.8/http-tests#assert-location)                                          | [assertViewHasAll](https://laravel.com/docs/5.8/http-tests#assert-view-has-all)                            |
| [assertNotFound](https://laravel.com/docs/5.8/http-tests#assert-not-found)                                         | [assertViewIs](https://laravel.com/docs/5.8/http-tests#assert-view-is)                                     |
| [assertOk](https://laravel.com/docs/5.8/http-tests#assert-ok)                                                      | [assertViewMissing](https://laravel.com/docs/5.8/http-tests#assert-view-missing)                           |

**assertCookie**

断言响应包含给定的 cookie：

```php
$response->assertCookie($cookieName, $value = null);
```

***

**assertCookieExpired**

断言响应包含给定的 cookie 并且它已过期：

```php
$response->assertCookieExpired($cookieName);
```

***

**assertCookieNotExpired**

断言响应包含给定的 cookie 并且它没有过期：

```php
$response->assertCookieNotExpired($cookieName);
```

***

**assertCookieMissing**

断言响应不包含给定的 cookie：

```php
$response->assertCookieMissing($cookieName);
```

***

**assertDontSee**

断言给定字符串不包含在响应中：

```php
$response->assertDontSee($value);
```

***

**assertDontSeeText**

断言给定字符串不包含在响应文本中：

```php
$response->assertDontSeeText($value);
```

***

**assertExactJson**

断言响应包含给定 JSON 数据的完全匹配：

```php
$response->assertExactJson(array $data);
```

***

**assertForbidden**

断言响应具有禁止的状态代码：

```php
$response->assertForbidden();
```

***

**assertHeader**

断言响应中存在给定标头：

```php
$response->assertHeader($headerName, $value = null);
```

***

**assertHeaderMissing**

断言响应中不存在给定标头：

```php
$response->assertHeaderMissing($headerName);
```

***

**assertJson**

断言响应包含给定的 JSON 数据：

```php
$response->assertJson(array $data);
```

***

**assertJsonCount**

断言响应 JSON 有一个数组，其中包含给定键的预期项数：

```php
$response->assertJsonCount($count, $key = null);
```

***

**assertJsonFragment**

断言响应包含给定的 JSON 片段：

```php
$response->assertJsonFragment(array $data);
```

***

**assertJsonMissing**

断言响应不包含给定的 JSON 片段：

```php
$response->assertJsonMissing(array $data);
```

***

**assertJsonMissingExact**

断言响应不包含确切的 JSON 片段：

```php
$response->assertJsonMissingExact(array $data);
```

***

**assertJsonMissingValidationErrors**

断言响应没有给定键的 JSON 验证错误：

```php
$response->assertJsonMissingValidationErrors($keys);
```

***

**assertJsonStructure**

断言响应具有给定的 JSON 结构：

```php
$response->assertJsonStructure(array $structure);
```

***

**assertJsonValidationErrors**

断言响应具有给定的 JSON 验证错误：

```php
$response->assertJsonValidationErrors(array $data);
```

***

**assertLocation**

断言响应在 `Location` 头中具有给定的 URI 值：

```php
$response->assertLocation($uri);
```

***

**assertNotFound**

断言响应有一个未找到的状态代码：

```php
$response->assertNotFound();
```

***

**assertOk**

断言响应具有200状态代码：

```php
$response->assertOk();
```

***

**assertPlainCookie**

断言响应包含给定的 cookie（未加密）：

```php
$response->assertPlainCookie($cookieName, $value = null);
```

***

**assertRedirect**

断言响应是一个重定向到给定的 URI：

```php
$response->assertRedirect($uri);
```

***

**assertSee**

断言给定字符串包含在响应中：

```php
$response->assertSee($value);
```

***

**assertSeeInOrder**

断言给定字符串按顺序包含在响应中：

```php
$response->assertSeeInOrder(array $values);
```

***

**assertSeeText**

断言给定字符串包含在响应文本中：

```php
$response->assertSeeText($value);
```

***

**assertSeeTextInOrder**

断言给定字符串在响应文本中按顺序包含：

```php
$response->assertSeeTextInOrder(array $values);
```

***

**assertSessionHas**

断言会话包含给定的数据段：

```php
$response->assertSessionHas($key, $value = null);
```

***

**assertSessionHasInput**

断言会话在闪存输入数组中具有给定值：

```php
$response->assertSessionHasInput($key, $value = null);
```

***

**assertSessionHasAll**

断言会话具有给定的值列表：

```php
$response->assertSessionHasAll(array $data);
```

***

**assertSessionHasErrors**

断言会话包含给定字段的错误：

```php
$response->assertSessionHasErrors(array $keys, $format = null, $errorBag = 'default');
```

***

**assertSessionHasErrorsIn**

断言会话有给定的错误：

```php
$response->assertSessionHasErrorsIn($errorBag, $keys = [], $format = null);
```

***

**assertSessionHasNoErrors**

断言会话没有错误：

```php
$response->assertSessionHasNoErrors();
```

***

**assertSessionDoesntHaveErrors**

断言会话对于给定的键没有错误：

```php
$response->assertSessionDoesntHaveErrors($keys = [], $format = null, $errorBag = 'default');
```

***

**assertSessionMissing**

断言会话没有包含给定的键：

```php
$response->assertSessionMissing($key);
```

***

**assertStatus**

断言响应有一个给定的状态码：

```php
$response->assertStatus($code);
```

***

**assertSuccessful**

断言响应有一个成功（200）的状态码：

```php
$response->assertSuccessful();
```

***

**assertUnauthorized**

断言响应有一个未授权（401）的状态码：

```php
$response->assertUnauthorized();
```

***

**assertViewHas**

断言响应视图是给定的一段数据：

```php
$response->assertViewHas($key, $value = null);
```

***

**assertViewHasAll**

断言响应视图有一个给定的数据列表：

```php
$response->assertViewHasAll(array $data);
```

***

**assertViewIs**

断言给定视图是通过路由返回的：

```php
$response->assertViewIs($value);
```

***

**assertViewMissing**

断言响应视图缺少一段绑定的数据：

```php
$response->assertViewMissing($key);
```

***

### 认证断言

Laravel 还为你的 [PHPUnit](https://phpunit.de) 测试提供了各种与认证相关的断言:

| 方法 | 描述 |
| --- | --- |
| `$this->assertAuthenticated($guard = null);` | 断言用户已被认证 |
| `$this->assertGuest($guard = null);` | 断言用户未被认证 |
| `$this->assertAuthenticatedAs($user, $guard = null);` | 断言给定的用户已被认证 |
| `$this->assertCredentials(array $credentials, $guard = null);` | 断言给定的凭据有效 |
| `$this->assertInvalidCredentials(array $credentials, $guard = null);` | 断言给定的凭据无效 |
