# 验证

## 简介

Laravel 提供几个不同的方式去验证你的应用程序传入的数据。默认情况下，Laravel 的基控制器类使用 `ValidatesRequests` 特性，该特性提供一种方便的方法以各种强大的验证规则去验证传入的 HTTP 请求。

## 验证快速入门

要了解关于 Laravel 的强大验证功能，让我们看一个验证表单并将错误消息显示回用户的完整示例。

### 定义路由

首先，让我们假设我们有如下路由定义在我们的 `routes/web.php` 文件：

```php
Route::get('post/create', 'PostController@create');

Route::post('post', 'PostController@store');
```

`GET` 路由将显示用户创建的一个新博客文章的表单，而 `POST` 路由将存储新博客文章到数据库。

### 创建控制器

接下来，我们来看一个处理这些路由的简单控制器。我们现在将 `store` 方法留空：

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use App\Http\Controllers\Controller;

class PostController extends Controller
{
    /**
     * 展示表单去创建一个新博客文章。
     *
     * @return Response
     */
    public function create()
    {
        return view('post.create');
    }

    /**
     * 存储一个新博客文章。
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        // 验证和存储博客文章...
    }
}
```

### 编写验证逻辑

现在我们准备用逻辑来填充我们的 `store` 方法去验证新的博客文章。要这样做，我们将使用 `Illuminate\Http\Request` 对象提供的 `validate` 方法。如果验证规则通过，你的代码将继续正常执行；但是，如果验证失败，将抛出一个异常并且合适的错误响应将自动发送回用户。在这种传统的 HTTP 请求情况下，将生成一个重定向响应，同时为 AJAX 请求发送一个 JSON 响应。

为了更好地理解 `validate` 方法，让我们跳回到 `store` 方法：

```php
/**
 * 存储一个新的博客文章。
 *
 * @param  Request  $request
 * @return Response
 */
public function store(Request $request)
{
    $validatedData = $request->validate([
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
    ]);

    // 博客文章验证通过...
}
```

如你所见，我们传递期望的验证规则到 `validate` 方法。同样，如果验证失败，合适的响应将自动生成。如果验证通过，我们的控制器将继续正常执行。

### 停止首次验证失败

有时我们希望第一次验证失败后对属性停止运行验证规则。为此，分配 `bail` 规则到属性：

```php
$request->validate([
    'title' => 'bail|required|unique:posts|max:255',
    'body' => 'required',
]);
```

在该示例中，如果 `title` 属性的 `unique` 规则验证失败，`max` 规则将不被检查。规则将按它们分配的顺序验证。

#### 关于嵌套属性的注意事项

如果你的 HTTP 请求包含『嵌套』参数，你可以在你的验证规则中使用『点』语法来指定它们：

```php
$request->validate([
    'title' => 'required|unique:posts|max:255',
    'author.name' => 'required',
    'author.description' => 'required',
]);
```

### 显示验证错误

那么，如果传入的请求参数没有通过验证规则该怎么办？如前所述，Laravel 将自动重定向用户回到他们之前的位置。此外，所有的验证错误将自动 [闪存到会话](https://laravel.com/docs/5.8/session#flash-data)。

再次，注意我们不必显式绑定错误消息到 `GET` 路由的视图。这是因为 Laravel 将检查会话数据中的错误，并自动绑定它们到视图（如果可用）。`$errors` 变量是 `Illuminate\Support\MessageBag`的一个实例。有关使用此对象的更多信息，[查看其文档](https://laravel.com/docs/5.8/validation#working-with-error-messages)。

{% hint style="info" %}

`$errors` 变量通过 `Illuminate\View\Middleware\ShareErrorsFromSession` 中间件绑定到视图，它通过 `web` 中间件组提供。**当此中间件被应用时，在你的视图中一个 `$errors` 变量将总是可用**，允许你方便地假定 `$errors` 变量始终定义并可以安全使用。

{% endhint %}

因此，在我们的示例中，当验证失败时，用户将被重定向到我们的控制器的 `create` 方法，允许我们在视图中去显示错误消息：

```php
<!-- /resources/views/post/create.blade.php -->

<h1>创建文章</h1>

@if ($errors->any())
    <div class="alert alert-danger">
        <ul>
            @foreach ($errors->all() as $error)
                <li>{{ $error }}</li>
            @endforeach
        </ul>
    </div>
@endif

<!-- 创建文章表单 -->
```

### 关于可选字段的注意事项

默认情况下，Laravel 在你的应用程序的全局中间件堆栈中包含 `TrimStrings` 和 `ConvertEmptyStringsToNull` 中间件。这些中间件通过 `App\Http\Kernel` 类列在堆栈中。正因如此，如果你不想验证程序将 `null` 值视为无效，则通常需要去标记你的『可选的』请求字段作为 `nullable`。例如：

```php
$request->validate([
    'title' => 'required|unique:posts|max:255',
    'body' => 'required',
    'publish_at' => 'nullable|date',
]);
```

在这个实例中，我们指定 `publish_at` 字段即可为 `null` 也可为有效的日期表示。如果 `nullable` 修饰符没有添加到规则定义中，验证程序将 `null` 视为无效日期。

#### AJAX 请求 & 验证

在此实例中，我们使用传统的表单去发送数据到应用程序。然而，许多应用程序使用 AJAX 请求。当在一个 AJAX 请求期间使用 `validate` 方法，Laravel 将不生成一个重定向响应。相反，Laravel 生成一个包含所有验证错误的 JSON 响应。此 JSON 响应将与一个 422 HTTP 状态码一并发送。

## 表单请求验证

### 创建表单请求

对于更复杂的验证场景，你可能希望去创建一个『表单请求』。表单请求是包含验证逻辑的自定义请求类。要创建一个表单请求类，使用 `make:request` Artisan CLI 命令：

```php
php artisan make:request StoreBlogPost
```

生成的类将放置在 `app/Http/Requests` 目录。如果此目录不存在，当你运行 `make:request` 命令时将创建。让我们添加一些验证规则到 `rules` 方法：

```php
/**
 * 获取应用到请求的验证规则。
 *
 * @return array
 */
public function rules()
{
    return [
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
    ];
}
```

{% hint style="info" %}

你可以在 `rules` 方法的签名上键入类型提示所需的任何依赖项。它们将通过 Laravel [服务容器](https://laravel.com/docs/5.8/container) 自动解析。

{% endhint %}

那么，如何评估验证规则呢？所有你需要做的就是在你的控制器方法上键入类型提示请求。在调用控制器的方法之前验证传入的表单请求，意味着你不需要你使用任何验证逻辑掺杂到你的控制器中：

```php
/**
 * 存储传入的博客文章。
 *
 * @param  StoreBlogPost  $request
 * @return Response
 */
public function store(StoreBlogPost $request)
{
    // 传入的请求通过验证...

    // 检索通过验证的输入数据...
    $validated = $request->validated();
}
```

如果验证失败，生成的重定向响应发送使用户回到到他们之前的位置。错误也将闪存到会话以便于显示可用。如果请求是一个 AJAX 请求，一个 422 状态码的 HTTP 请求将返回到用户，包括验证错误的 JSON 表示。

#### 添加表单请求后钩子

如果你想在表单请求中添加一个『之后』钩子，你可以使用 `withValidator` 方法。此方法接受完全构造的验证器，允许你在实际评估验证规则之前调用其任何方法：

```php
/**
 * Configure the validator instance.
 *
 * @param  \Illuminate\Validation\Validator  $validator
 * @return void
 */
public function withValidator($validator)
{
    $validator->after(function ($validator) {
        if ($this->somethingElseIsInvalid()) {
            $validator->errors()->add('field', 'Something is wrong with this field!');
        }
    });
}
```

### 授权表单请求

表单请求类还包含一个 `authorize` 方法。在这个方法中，你可以检查认证的用户是否实际有更新一个给定资源的权限。例如，你可以确定一个用户是否实际拥有一个他们试图去更新博客评论的权限：

```php
/**
 * 确定用户是否有权限发出此请求。
 *
 * @return bool
 */
public function authorize()
{
    $comment = Comment::find($this->route('comment'));

    return $comment && $this->user()->can('update', $comment);
}
```

由于所有表单请求继承基础 Laravel 请求类，我们可以使用 `user` 方法去访问当前认证的用户。也注意上面示例中对 `route` 方法的调用。此方法允许你访问在被调用路由上定义的 URI 参数，比如：在下面实例的 `{comment}` 参数：

```php
Route::post('comment/{comment}');
```

如果 `authorize` 方法返回 `false`，一个 403 状态码的 HTTP 响应将自动返回，并且你的控制器方法将不执行：

如果你计划在你应用程序的另一部分中有授权逻辑，从 `authorize` 方法返回 `true`：

```php
/**
 * 确定用户是否被授权发出此请求。
 *
 * @return bool
 */
public function authorize()
{
    return true;
}
```

{% hint style="info" %}

你可以在 `authorize` 方法的签名中键入类型提示你需要的任何依赖项。它们将自动通过 [服务容器](https://laravel.com/docs/5.8/container) 自动解析。

{% endhint %}

### 定制错误消息

你可以通过覆盖 `messages` 方法来自定义表单请求使用的错误消息。此方法应当返回一个属性 / 规则键值对的数组以及相对应的错误消息：

```php
/**
 * 获取已定义验证规则的错误消息。
 *
 * @return array
 */
public function messages()
{
    return [
        'title.required' => 'A title is required',
        'body.required'  => 'A message is required',
    ];
}
```

### 自定义验证属性

如果你想把你的验证消息的 `:attribute` 部分替换为一个自定义属性名称，你可以通过覆盖 `attributes` 方法指定自定义名称。此方法应当返回一个属性 / 名称键值对的数组：

```php
/**
 * 获取验证器错误的自定义属性。
 *
 * @return array
 */
public function attributes()
{
    return [
        'email' => 'email address',
    ];
}
```

## 手动创建验证器

如果你不想在请求上使用 `validate` 方法，你可以使用 `Validator` [facade](https://laravel.com/docs/5.8/facades) 手动创建一个验证器实例。facade 上的 `make` 方法生成一个新的验证器实例：

```php
<?php

namespace App\Http\Controllers;

use Validator;
use Illuminate\Http\Request;
use App\Http\Controllers\Controller;

class PostController extends Controller
{
    /**
     * 存储一个新博客文章。
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        $validator = Validator::make($request->all(), [
            'title' => 'required|unique:posts|max:255',
            'body' => 'required',
        ]);

        if ($validator->fails()) {
            return redirect('post/create')
                        ->withErrors($validator)
                        ->withInput();
        }

        // 存储博客文章...
    }
}
```

传递到 `make` 方法的第一个参数是验证中的数据。第二个参数是应当应用于数据的验证规则。

检查请求验证是否失败之后，你可以使用 `withErrors` 方法去闪存错误消息到会话。当使用此方法时，`$errors` 变量将自动共享到重定向后的视图，允许你去轻松把它们显示回用户。`withErrors` 方法接受一个验证器，一个 `MessageBag` 或一个 PHP 数组。

### 自动重定向

如果你想手动创建一个验证器实例，但仍然利用请求的 `validate` 方法提供的自动重定向，你可以在现有的验证器实例上调用 `validate` 方法。如果验证失败，用户将自动重定向，或者在一个 AJAX 请求情形下，将返回一个 JSON 响应：

```php
Validator::make($request->all(), [
    'title' => 'required|unique:posts|max:255',
    'body' => 'required',
])->validate();
```

### 命名错误包

如果在单页中你有多个表单，你可能希望命名错误的 `MessageBag`，允许你检索指定表单的错误消息。将名称作为第二个参数传递给 `withErrors`：

```php
return redirect('register')
            ->withErrors($validator, 'login');
```

接下来你可以从 `$errors` 变量中访问命名的 `MessageBag` 实例：

```php
{{ $errors->login->first('email') }}
```

### 验证后钩子

验证器还允许你附加验证完成后运行的回调。这允许你轻松地执行进一步验证并甚至添加更多错误消息到消息收集器中。首先，在验证器实例上使用 `after` 方法：

```php
$validator = Validator::make(...);

$validator->after(function ($validator) {
    if ($this->somethingElseIsInvalid()) {
        $validator->errors()->add('field', 'Something is wrong with this field!');
    }
});

if ($validator->fails()) {
    //
}
```

## 处理错误消息

在 `Validator` 实例上调用 `errors` 方法之后，你将接受到一个 `Illuminate\Support\MessageBag` 实例，该实例具有处理错误消息的各种方便的方法。自动提供给所有视图的 `$errors` 变量也是 `MessageBag` 类的一个实例。

- **检索字段的第一条错误消息**

为了检索给定字段的第一条错误消息，使用 `first` 方法：

```php
$errors = $validator->errors();

echo $errors->first('email');
```

- **检索字段的所有错误消息**

如果你需要检索一个给定字段的所有消息数组，使用 `get` 方法：

```php
foreach ($errors->get('email') as $message) {
    //
}
```

如果你要验证数组表单字段，你可以使用 `*` 字符检索每个数组元素的所有消息：

```php
foreach ($errors->get('attachments.*') as $message) {
    //
}
```

- **检索所有字段的所有错误消息**

要检索所有字段的所消息数组，使用 `all` 方法：

```php
foreach ($errors->all() as $message) {
    //
}
```

- **确定字段是否存在消息**

`has` 方法可以用于确定一个给定的字段是否存在任何错误消息：

```php
if ($errors->has('email')) {
    //
}
```

### 自定义错误消息

如果需要，你可以使用自定义错误消息进行验证而不是默认值。有几种方法去指定自定义消息。首先，你可以将自定义消息作为第三个参数传递到 `Validator:make` 方法：

```php
$messages = [
    'required' => 'The :attribute field is required.',
];

$validator = Validator::make($input, $rules, $messages);
```

在这个实例中，`:attribute` 占位符将通过实际的验证中的字段名称替换。你还可以在验证消息中利用其它占位符。例如：

```php
$messages = [
    'same'    => 'The :attribute and :other must match.',
    'size'    => 'The :attribute must be exactly :size.',
    'between' => 'The :attribute value :input is not between :min - :max.',
    'in'      => 'The :attribute must be one of the following types: :values',
];
```

#### 为给定属性指定自定义消息

有时你可能希望仅为一个特定的字段去指定一个自定义的错误消息。你可以使用『点』表示法来这样做。首先指定属性的名称，其次是规则：

```php
$messages = [
    'email.required' => 'We need to know your e-mail address!',
];
```

#### 在语言文件中指定自定义消息

在大多数情况下，你可能会在语言文件中指定自定义消息，而不是直接将它们传递到 `Validator`。为此，在 `resources/lang/xx/validation.php` 文件中添加你的消息到 `custom` 数组：

```php
'custom' => [
    'email' => [
        'required' => 'We need to know your e-mail address!',
    ],
],
```

#### 在语言文件中指定自定义属性

如果你希望你的验证消息的 `:attribute` 部分替换为自定义属性名称，你可以在你的 `resources/lang/xx/validation.php` 语言文件的 `attribute` 数组中指定自定义名称：

```php
'attributes' => [
    'email' => 'email address',
],
```

#### 在语言文件中指定自定义值

有时你可能需要你的验证消息的 `:value` 部分替换为值的自定义表示。例如，考虑如下的规则，如果 `payment_type` 的值为 `cc` 时该规则指定必须有一个信用卡号：

```php
$request->validate([
    'credit_card_number' => 'required_if:payment_type,cc'
]);
```

如果此验证规则失败，它将产生如下错误消息：

```bash
支付类型为 cc 时信用卡号字段必须填写。
```

你可以在你的 `validation` 语言文件中通过定义一个 `values` 数组指定一个自定义值表示，而不是将 `cc` 显示为付款类型值：

```php
'values' => [
    'payment_type' => [
        'cc' => 'credit card'
    ],
],
```

现在，如果验证规则失败，它将产生如下消息：

```bash
支付类型为信用卡时信用卡号字段必须填写。
```

## 可用的验证规则

以下是所有可用验证规则及其它们功能的列表：

|                                                                                       |                                                                                   |                                                                                           |
| ------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------- |
| [Accepted](https://laravel.com/docs/5.8/validation#rule-accepted)                     | [Distince](https://laravel.com/docs/5.8/validation#rule-distinct)                 | [Nullable](https://laravel.com/docs/5.8/validation#rule-nullable)                         |
| [Active URL](https://laravel.com/docs/5.8/validation#rule-active-url)                 | [E-Mail](https://laravel.com/docs/5.8/validation#rule-email)                      | [Numeric](https://laravel.com/docs/5.8/validation#rule-numeric)                           |
| [After(Date)](https://laravel.com/docs/5.8/validation#rule-after)                     | [Exists(Database)](https://laravel.com/docs/5.8/validation#rule-exists)           | [Present](https://laravel.com/docs/5.8/validation#rule-present)                           |
| [After Or Equal(Date)](https://laravel.com/docs/5.8/validation#rule-after-or-equal)   | [File](https://laravel.com/docs/5.8/validation#rule-file)                         | [Regular Expression](https://laravel.com/docs/5.8/validation#rule-regex)                  |
| [Alpha](https://laravel.com/docs/5.8/validation#rule-alpha)                           | [Filled](https://laravel.com/docs/5.8/validation#rule-filled)                     | [Required](https://laravel.com/docs/5.8/validation#rule-required)                         |
| [Alpha Dash](https://laravel.com/docs/5.8/validation#rule-alpha-dash)                 | [Greater Than](https://laravel.com/docs/5.8/validation#rule-gt)                   | [Required If](https://laravel.com/docs/5.8/validation#rule-required-if)                   |
| [Alpha Numeric](https://laravel.com/docs/5.8/validation#rule-alpha-num)               | [Greater Than Or Equal](https://laravel.com/docs/5.8/validation#rule-gte)         | [Required Unless](https://laravel.com/docs/5.8/validation#rule-required-unless)           |
| [Array](https://laravel.com/docs/5.8/validation#rule-array)                           | [Image(File)](https://laravel.com/docs/5.8/validation#rule-image)                 | [Required With](https://laravel.com/docs/5.8/validation#rule-required-without)            |
| [Bail](https://laravel.com/docs/5.8/validation#rule-bail)                             | [In](https://laravel.com/docs/5.8/validation#rule-in)                             | [Required With All](https://laravel.com/docs/5.8/validation#rule-required-with-all)       |
| [Before(Date)](https://laravel.com/docs/5.8/validation#rule-before)                   | [In Array](https://laravel.com/docs/5.8/validation#rule-in-array)                 | [Required Without](https://laravel.com/docs/5.8/validation#rule-required-without)         |
| [Before Or Equal(Date)](https://laravel.com/docs/5.8/validation#rule-before-or-equal) | [Integer](https://laravel.com/docs/5.8/validation#rule-integer)                   | [Required Without All](https://laravel.com/docs/5.8/validation#rule-required-without-all) |
| [Between](https://laravel.com/docs/5.8/validation#rule-between)                       | [IP Address](https://laravel.com/docs/5.8/validation#rule-ip)                     | [Same](https://laravel.com/docs/5.8/validation#rule-same)                                 |
| [Boolean](https://laravel.com/docs/5.8/validation#rule-boolean)                       | [JSON](https://laravel.com/docs/5.8/validation#rule-json)                         | [Size](https://laravel.com/docs/5.8/validation#rule-size)                                 |
| [Confirmed](https://laravel.com/docs/5.8/validation#rule-confirmed)                   | [Less Than](https://laravel.com/docs/5.8/validation#rule-lt)                      | [Starts With](https://laravel.com/docs/5.8/validation#rule-starts-with)                   |
| [Date](https://laravel.com/docs/5.8/validation#rule-date)                             | [Less Than Or Equal](https://laravel.com/docs/5.8/validation#rule-lte)            | [String](https://laravel.com/docs/5.8/validation#rule-string)                             |
| [Date Equals](https://laravel.com/docs/5.8/validation#rule-date-equals)               | [Max](https://laravel.com/docs/5.8/validation#rule-max)                           | [Timezone](https://laravel.com/docs/5.8/validation#rule-timezone)                         |
| [Date Format](https://laravel.com/docs/5.8/validation#rule-date-format)               | [MIME Types](https://laravel.com/docs/5.8/validation#rule-mimetypes)              | [Unique(Database)](https://laravel.com/docs/5.8/validation#rule-unique)                   |
| [Different](https://laravel.com/docs/5.8/validation#rule-different)                   | [MIME Type By File Extension](https://laravel.com/docs/5.8/validation#rule-mimes) | [URL](https://laravel.com/docs/5.8/validation#rule-url)                                   |
| [Digits](https://laravel.com/docs/5.8/validation#rule-digits)                         | [Min](https://laravel.com/docs/5.8/validation#rule-min)                           | [UUID](https://laravel.com/docs/5.8/validation#rule-uuid)                                 |
| [Digits Between](https://laravel.com/docs/5.8/validation#rule-digits-between)         | [NotIn](https://laravel.com/docs/5.8/validation#rule-not-in)                      |
| [Dimensions(Image Files)](https://laravel.com/docs/5.8/validation#rule-dimensions)    | [Not Regex](https://laravel.com/docs/5.8/validation#rule-not-regex)               |

### accepted

验证字段必须为 _yes_，_on_，_1_ 或 _true_。这有助于验证『服务条款』的接受程度。

### active_url

根据 PHP 的 `dns_get_record` 函数，验证字段必须为一个有效的 A 或者 AAAA 记录。

### after:date

验证中的字段必须是给定日期以后的值。该日期将被传递到 PHP 的 `strtotime` 函数中:

```php
'start_date' => 'required|date|after:tomorrow'
```

你可以指定另一个与日期进行比较的字段，而不是传递通过 `strtotime` 评估的日期字符串：

```php
'finish_date' => 'required|date|after:start_date'
```

### after_or_equal:date

验证字段必须是一个给定日期之后或等于给定日期的值。有关更多信息，看 [之后](https://laravel.com/docs/5.8/validation#rule-after) 规则。

### alpha

验证字段必须完全是字母字符。

### alpha_dash

验证字段可以包含字母数字字符以及中划线和下划线。

### alpha_num

验证字段必须完全是字母数字字符。

### array

验证中的字段必须是 PHP 数组。

### bail

第一次验证失败后停止运行验证规则。

### before:date

验证中的字段必须是给定日期之前的一个值。日期将被传递到 PHP 的 `strtotime` 函数中。此外，像 [之后](https://laravel.com/docs/5.8/validation#rule-after) 规则一样，验证的另一个字段的名称可以作为 `date` 值提供。

### before_or_equal:date

验证字段必须是一个给定日期之前的或者等于给定日期的值。该日期将传递到 PHP 的 `strtotime` 函数。此外，像 [之后](https://laravel.com/docs/5.8/validation#rule-after) 规则一样，验证的另一个字段的名称可以作为 `date` 值提供。

### between:min,max

验证字段的大小必须在给定的 *min* 和 *max* 之间。字符串，数字，数组和文件的计算方式与 `size` 规则相同。

### boolean

验证字段必须能够转换为一个布尔值。接受的输入是 `true`，`false`，`1`，`0`，`"1"`，`"0"`。

### confirmed

验证的字段必须具有 `foo_confirmation` 字段的匹配。例如，如果验证字段是 `password`，在输入中必须存在一个匹配的 `password_confirmation` 字段。

### date

根据 PHP 的 `strtotime` 函数，验证字段必须是一个有效的非相对日期。

### date_equals:date

验证字段必须等于给定的日期。日期将传递到 PHP 的 `strtotime` 函数。

### date_format:format

验证字段必须与给定的格式匹配。当验证一个字段时，你应该即可以使用 `date` 也可以使用 `date_format`，而不是两者都使用。

### different:field

验证字段必须具有与 *field* 不同的值。

### digits:value

验证字段必须是数字，并且具有确切的值长度。

### digits_between:min,max

验证字段的长度必须介于给定的最大值和最小值之间。

### dimensions

验证中的字段必须是图片并满足符合规则参数指定的约束尺寸：

```php
'avatar' => 'dimensions:min_width=100,min_height=200'
```

可用的约束有：`min_width`，`max_width`，`min_height`，`max_height`，`width`，`height`，`ratio`。

约束比例应该表示为宽除以高。该约束比例可以指定为一个像 `3/2` 的语句或者一个像 `1.5` 的浮点数：

```php
'avatar' => 'dimensions:ratio=3/2'
```

由于此规则要求几个参数，你可以使用 `Rule::dimensions` 方法流畅地构造规则：

```php
use Illuminate\Validation\Rule;

Validator::make($data, [
    'avatar' => [
        'required',
        Rule::dimensions()->maxWidth(1000)->maxHeight(500)->ratio(3 / 2),
    ],
]);
```

### distinct

当使用数组时，验证字段必须没有任何重复的值。

```php
'foo.*.id' => 'distinct'
```

### email

验证字段必须是一个格式化的电子邮件地址。

### exists:table,column

验证字段必须存在于一个给定的数据表中。

#### 存在规则的基本用法

```php
'state' => 'exists:states'
```

如果 `column` 选项没有指定，则将使用字段名称。

#### 指定自定义列名

```php
'state' => 'exists:states,abbreviation'
```

偶尔，你可能需要去指定用于 `exists` 查询的一个特定的数据库连接。你可以使用『点』语法将连接名称添加到表名称前来完成此操作：

```php
'email' => 'exists:connection.staff,email'
```

如果你希望通过验证规则去自定义执行的查询，你可以使用 `Rule` 类去流畅定义规则。在这个实例中，我们也指定验证规则作为一个数组而不是使用 `|` 字符去分隔它们：

```php
use Illuminate\Validation\Rule;

Validator::make($data, [
    'email' => [
        'required',
        Rule::exists('staff')->where(function ($query) {
            $query->where('account_id', 1);
        }),
    ],
]);
```

### file

验证的字段必须是一个成功上传的文件。

### filled

验证的字段存在时必须不能为空。

### gt:field

验证的字段必须大于给定的字段。两个字段必须具有相同的类型。字符串，数字，数组和文件使用与 `size` 规则相同的约束进行计算。

### gte:field

验证字段必须大于或者等于给定的字段。两个字段必须具有相同的类型。字符串，数字，数组和文件使用与 `size` 规则相同的约束进行计算。

### image

验证字段必须是一个图像（jpeg，png，bmp，gif 或 svg）。

### in:foo,bar,...

验证字段必须包含在一个给定的值列表中。由于此规则通常要求你 `implode` 一个数组，`Rule::in` 方法可以用于流畅地构造规则 ：

```php
use Illuminate\Validation\Rule;

Validator::make($data, [
    'zones' => [
        'required',
        Rule::in(['first-zone', 'second-zone']),
    ],
]);
```

### in_array:anotherfield.*

验证字段必须存在于别一个字段的值中。

### integer

验证字段必须是一个整数。

### ip

验证字段必须是一个 IP 地址。

### ipv4

验证字段必须是一个 IPv4 地址。

### ipv6

验证字是必须一个 IPv6 地址。

### json

验证字段必须是有效的 JSON 字符串。

### lt:field

验证字段必须小于给定的字段。两个字段必须具有相同的类型。字符串，数字，数组和文件使用与 `size` 规则相同的约束进行计算。

### lte:field

验证字段必须小于或等于给定的字段。两个字段必须具有相同的类型。字符串，数字，数组和文件使用与 `size` 规则相同的约束进行计算。

### max:value

验证规则必须小于或等于最大值。字符串，数字，数组和文件的计算方式与 `size` 规则相同。

### mimetypes:text/plain,...

验证字段必须匹配给定 MIME 类型中的一种：

```php
'video' => 'mimetypes:video/avi,video/mpeg,video/quicktime'
```

要确定上传文件的 MIME 类型，将读取文件的内容，框架将尝试猜测 MIME 类型，这可以与客户端提供的 MIME 类型不一样。

### mimes:foo,bar,...

验证的字段必须有一个与列出的扩展名之一相对应的 MIME 类型。

#### MIME 规则的基本用法

```php
'photo' => 'mimes:jpeg,bmp,png'
```

即使你仅需要去指定扩展，此规则实际上验证文件的 MIME 类型是通过读取文件的内容并猜测它的 MIME 类型。

可以在以下的位置找到一个完整的 MIME 类型及其它们对应的扩展列表：`https://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types`。

### min:value

验证的字段必须有一个最小值。字符串，数字，数组和文件的计算方式与 `size` 规则相同。

### not_in:foo,bar,...

验证的字段必须不包含在给定的值列表中。`Rule::notIn` 方法可用于流畅地构造规则:

```php
use Illuminate\Validation\Rule;

Validator::make($data, [
    'toppings' => [
        'required',
        Rule::notIn(['sprinkles', 'cherries']),
    ],
]);
```

### not_regex:pattern

验证字段必须与给定的正则表达式不匹配。

在内部，此规则使用 PHP 的 `preg_match` 函数。指定的模式应该遵循 `preg_match` 所需的相同格式，并因此还包含有效的分隔符。例如：`'email' => 'not_regex:/^.+$/i'`。

**注意：**当使用 `regex` / `not_regex` 模式时，它可能需要在数组中指定规则而不是使用管道分隔符，尤其是正则表达式包含一个管道符时。

### nullable

验证字段可以为 `null`。当验证基础类型（如可以包含 `null` 值的字符串和数字）时特别有用。

### numeric

验证字段必须是数字。

### present

验证字段必须在输入数据中存在，但可以为空。

### regex:pattern

验证字段必须匹配给定的正则表达式。

在内部，此规则使用 PHP 的 `preg_match` 函数。指定的模式应该遵循 `preg_match` 所需的相同格式，并因此还包含有效的分隔符。例如：`'email' => 'regex:/^.+@.+$/i'`。

**注意**：当使用 `regex` / `not_regex` 模式时，它可能需要在数组中指定规则而不是使用管道分隔符，尤其是正则表达式包含一个管道符时。

### required

验证字段必须在输入数据中存在且不为空。如果以下条件之一为真，则一个字段被认为为『空』：

- 值是 `null`；
- 值是一个空字符串；
- 值是一个空数组或者空 `Countable` 对象；
- 值是一个没有路径的上传文件。

### required_if:anotherfield,value,...

如果另一个字段等于任何值，则验证字段必须存在并且不为空。

如果你希望为 `required_if` 规则构造更多复杂条件，你可以使用 `Rule::requiredIf` 方法。此方法接受一个布尔或者一个闭包。当传递一个闭包时，闭包应当返回 `true` 或者 `false`，以指示验证字段是否是必需的：

```php
use Illuminate\Validation\Rule;

Validator::make($request->all(), [
    'role_id' => Rule::requiredIf($request->user()->is_admin),
]);

Validator::make($request->all(), [
    'role_id' => Rule::requiredIf(function () use ($request) {
        return $request->user()->is_admin;
    }),
]);
```

### required_unless:anotherfield,value,...

验证字段必须存在且不为空，除非另一个字段等于任何值。

### required_with:foo,bar,...

验证字段必须存在，并且仅在其它任何字段存在时才不为空。

### required_with_all:foo,bar,...

验证字段必须存在，并且仅在其它所有指定字段存在时才不为空。

### required_without:foo,bar,...

验证字段必须存在，并且仅当其它任何指定字段不存在时才不为空。

### required_without_all:foo,bar,...

验证字段必须存在，并且仅当其它所有指定字段不存在时才不为空。

### same:field

给定的字段必须匹配验证字段。

### size:value

验证字段必须有一个与给定值匹配的大小。对于字符串数据，值对应字符数。对于数值数据，值对应给定的整数值。对于数组，大小对应数组的 `count`。对于文件，大小对应以千字节为单位的文件大小。

### starts_with:foo,bar,...

验证字段必须从给定值中的一个开始。

### string

验证字段必须是一个字符串。如果你希望该字段也为 `null`，你应当分配 `nullable` 规则给该字段。

### timezone

验证字段必须是一个根据 PHP `timezone_identifiers_list` 函数的有效时区标识符。

### unique:table,column,except,idColumn

验证字段在给定的数据表中必须是唯一的。如果 `column` 选项不被指定，将使用字段名。

#### 指定一个自定义列名

```php
'email' => 'unique:users,email_address'
```

#### 自定义数据库连接

偶尔，你可能需要通过验证器为数据库查询设置一个自定义连接。如前所述，设置 `unique:users` 作为一个验证规则将使用默认的数据库连接去查询数据库。要覆盖这个，要使用『点』语法指定连接和表名：

```php
'email' => 'unique:connection.users,email_address'
```

#### 强制唯一规则而忽略给定的标识

有时，你可能希望在唯一检查期间去忽略给定的 ID。例如，考虑一个包含用户的姓名，电子邮件地址和位置的『更新个人资料』界面。你可能想去验证电子邮件是否唯一。然而，如果用户仅改变姓名字段并不更改电子邮件字段，你不想抛出验证错误，因为用户已经是电子邮件地址的所有者。

为了指示验证器去忽略用户的 ID，我们将使用 `Rule` 类流畅地定义规则。在本示例中，我们也将指定验证规则为一个数组而不是使用 `|` 字符去分隔规则：

```php
use Illuminate\Validation\Rule;

Validator::make($data, [
    'email' => [
        'required',
        Rule::unique('users')->ignore($user->id),
    ],
]);
```

{% hint style="danger" %}

你应当从不传递任何用户控制的请求输入到 `ignore` 方法。相反，你应当从一个 Eloquent 模型实例上仅传递一个系统生成的唯一 ID（如自增 ID 或 UUID）。否则，你的应用程序将易受 SQL 注入的攻击。

{% endhint %}

你可以传递整个模型实例，而不是传递模型键的值到 `ignore` 方法。Laravel 将自动从模型中提取该键：

```php
Rule::unique('users')->ignore($user)
```

如果你的表使用一个除 `id` 之外的主键列名，你可以在调用 `ignore` 方法时指定列名：

```php
Rule::unique('users')->ignore($user->id, 'user_id')
```

默认情况下，`unique` 规则将检查与验证的属性名称匹配的列的唯一性。然而，你可以把一个不同的列名作为第二个参数传递到 `unique` 方法：

```php
Rule::unique('users', 'email_address')->ignore($user->id),
```

#### 添加其它条件子句

你还可以使用 `where` 方法通过自定义查询指定额外查询约束。例如，让我们添加一个验证 `account_id` 为 `1` 的约束：

```php
'email' => Rule::unique('users')->where(function ($query) {
    return $query->where('account_id', 1);
})
```

### url

验证字段必须是一个有效的 URL。

### uuid

验证字段必须是一个有效的 RFC 4122（版本：1，3，4 或 5）通用唯一标识符（UUID）。

## 有条件地添加规则

### 存在时验证

在一些情形下，你可能希望 **仅** 在输入数组中存在字段时才去运行验证检查。要快速完成此任务，添加 `sometimes` 规则到你的规则列表中：

```php
$v = Validator::make($data, [
    'email' => 'sometimes|required|email',
]);
```

在上面的示例中，如果在 `$data` 数组中存在 `email` 字段时才将验证它。

{% hint style="info" %}

如果你企图去验证的字段始终存在，但可能是空的，查看 [关于可选字段的说明](https://laravel.com/docs/5.8/validation#a-note-on-optional-fields)。

{% endhint %}

### 复杂条件验证

```php
$v = Validator::make($data, [
    'email' => 'required|email',
    'games' => 'required|numeric',
]);
```

## 验证数组

## 自定义验证规则

### 使用规则对象

### 使用闭包

### 使用扩展

#### 定义错误信息

#### 隐式扩展

{% hint style="info" %}

{% endhint %}
