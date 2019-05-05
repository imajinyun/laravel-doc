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
