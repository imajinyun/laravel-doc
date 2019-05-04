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
