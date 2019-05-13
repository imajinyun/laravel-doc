# 本地化

## 简介

Laravel 的本地化特性为检索各种语言的字符串提供了一种方便的方式，允许你在应用程序中轻松支持多种语言。语言字符串存储在 `resources/lang` 目录中的文件中。在该目录中，应用程序支持的每种语言都应有一个子目录:

```php
/resources
    /lang
        /en
            messages.php
        /es
            messages.php
```

所有语言文件都返回一个键名的字符串数组。例如:

```php
<?php

return [
    'welcome' => 'Welcome to our application'
];
```

### 配置区域设置

应用程序的默认语言存储在 `config/app.php` 配置文件中。你可以修改此值以满足应用程序的需要。你还可以使用 `App` facade 上的 `setLocale` 方法在运行时更改活动语言：

```php
Route::get('welcome/{locale}', function ($locale) {
    App::setLocale($locale);

    //
});
```

你可以配置『后备语言』，当活动语言不包含给定的翻译字符串时将使用该语言。与默认语言一样，后备语言也在 `config/app.php` 配置文件中配置：

```php
'fallback_locale' => 'en',
```

**确定当前区域设置**

你可以在 `App` facade 上使用 `getLocale` 和 `isLocale` 方法去确定当前区域，或者检查该区域设置是否为一个给定的值：

```php
$locale = App::getLocale();

if (App::isLocale('en')) {
    //
}
```

## 定义翻译字符串

### 使用快捷键

通常，翻译字符串存储在 `resources/lang` 目录中的文件中。在该目录中，通过应用程序支持的每种语言都应有一个子目录：

```php
/resources
    /lang
        /en
            messages.php
        /es
            messages.php
```

所有语言文件都返回一个键名的字符串数组。例如：

```php
<?php

// resources/lang/en/messages.php

return [
    'welcome' => 'Welcome to our application'
];
```

### 使用翻译字符串作为键名

对于翻译要求很高的应用程序，在视图中引用字符串时，用一个『短键』定义每个字符串会很快变得混乱。为此，Laravel 还支持使用字符串的『默认』翻译作为键名来定义翻译字符串。

使用翻译字符串作为键名的翻译文件作为 JSON 文件存储在 `resources/lang` 目录中。例如，如果你的应用程序有一个西班牙语翻译，你应该创建一个 `resources/lang/es.json` 文件：

```php
{
    "I love programming.": "Me encanta programar."
}
```

## 检索翻译字符串

你可以使用 `__` 助手函数从语言文件中检索行。`__` 方法接受翻译字符串的文件和键名作为第的第一个参数。例如，让我们从 `resources/lang/messages.php` 语言文件中检索 `welcome` 翻译字符串：

```php
echo __('messages.welcome');

echo __('I love programming.');
```

如果你使用 [Blade 模版引擎](https://laravel.com/docs/5.8/blade)，你可以使用 `{{}}` 语法去回显翻译字符串或者使用 `@lang` 指令：

```php
{{ __('messages.welcome') }}

@lang('messages.welcome')
```

如果指定的翻译字符串不存在，`__` 函数将返回翻译字符串键名。因此，使用上面的实例，如果翻译字符串不存在，`__` 函数将返回 `messages.welcome`。

{% hint style="danger" %}

`@lang` 不转义任何输出。使用此指令时，你**完全负责**转义你自己输出。

{% endhint %}

### 替换翻译字符串中的参数

如果你愿意，你可以在翻译字符串中定义占位符。所有占位符都以 `:` 为前缀。例如，你可以使用占位符名称定义欢迎消息：

```php
'welcome' => 'Welcome, :name',
```

若要在检索翻译字符串时替换占位符，将替换数组作为第二个参数传递给 `__` 函数：

```php
echo __('messages.welcome', ['name' => 'dayle']);
```

如果你的占位符包含所有大写字母，或者仅将其首字母大写，则翻译后的值将相应地大写：

```php
'welcome' => 'Welcome, :NAME', // Welcome, DAYLE
'goodbye' => 'Goodbye, :Name', // Goodbye, Dayle
```

### 复数

复数是一个复杂的问题，因为不同的语言对复数有各种复杂的规则。通过使用『管道』符，可以区分字符串的单复数形式：

```php
'apples' => 'There is one apple|There are many apples',
```

你甚至可以创建更复杂的复数规则，为多个数字范围指定转翻译符串：

```php
'apples' => '{0} There are none|[1,19] There are some|[20,*] There are many',
```

定义具有复数选项的翻译字符串之后，你可以使用 `trans_choice` 函数去检索一个给定的『计数』的行。在本实例中，由于计数大于 1，因此返回翻译字符串的复数形式：

```php
echo trans_choice('messages.apples', 10);
```

你可以在复数字符串中定义占位符属性。这些占位符可以通过将数组作为第三个参数传递到 `trans_choice` 函数来替换：

```php
'minutes_ago' => '{1} :value minute ago|[2,*] :value minutes ago',

echo trans_choice('time.minutes_ago', 5, ['value' => 5]);
```

如果你想去显示传递到 `trans_choice` 函数的整型值，你可以使用 `:count` 占位符：

```php
'apples' => '{0} There are none|{1} There is one|[2,*] There are :count',
```

## 覆盖包语言文件

有些软件包可能会附带自己的语言文件。你可以将文件放在 `resources/lang/vendor/{package}/{locale}` 目录中，而不是更改包的核心文件来调整这些行。

因此，例如，如果你需要覆盖 `messages.php` 中命名为 `skyrim/hearthfire` 包的英语翻译字符串，你应该将语言文件放在：`resources/lang/vendor/hearthfire/en/messages.php` 文件中。在此文件中，你应该只定义要覆盖的翻译字符串。你没有覆盖的任何翻译字符串将从包的原始语言文件中加载。
