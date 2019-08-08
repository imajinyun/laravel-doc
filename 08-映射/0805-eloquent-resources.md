# Eloquent：API 资源

## 简介

构建 API 时，你可能需要一个位于你的 Eloquent 模型和实际返回给应用程序用户的 JSON 响应之间的转换层。Laravel 的资源类允许你富于表达力且轻松地将你的模型和模型集合转换为 JSON。

## 生成资源

要生成资源类，可以使用 `make:resource` Artisan 命令。默认情况下，资源将放在你的应用程序的 `app/Http/Resources` 目录中。资源继承了 `Illuminate\Http\Resources\Json\JsonResource` 类：

```bash
php artisan make:resource User
```

***

**资源集合**

除了生成转换单个模型的资源之外，你还可以生成负责转换模型集合的资源。这允许你的响应包括与给定资源的整个集合相关的链接和其他元信息。

要创建资源集合，你应当在创建资源时使用 `--collection` 标志。或者，在资源名称中包含单词 `Collection` 将指示 Laravel 它应该创建一个集合资源。集合资源继承了 `Illuminate\Http\Resources\Json\ResourceCollection` 类：

```bash
php artisan make:resource Users --collection

php artisan make:resource UserCollection
```

## 概念概述
