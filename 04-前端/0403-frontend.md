# JavaScript & CSS 脚手架

## 简介

虽然 Laravel 没有规定你使用哪种 JavaScript 或 CSS 预处理器，但它确实提供了使用 [Bootstrap](https://getbootstrap.com/) 和 [Vue](https://vuejs.org/) 的基本起点，这对许多应用程序都有帮助。默认情况下，Laravel 使用 [NPM](https://www.npmjs.com/) 来安装这两个前端软件包。

### CSS

[Laravel-Mix](https://laravel.com/docs/5.8/mix) 提供了一个干净，富有表现力的API，而不是编译 SASS 或 Less，这是纯 CSS 的扩展，增加了变量，mixins 和其他强大的功能，使得使用 CSS 变得更加愉快。在本文档中，我们将简要讨论 CSS 编译；但是，你应该查阅完整的 [Laravel Mix 文档](https://laravel.com/docs/5.8/mix)，以获取有关编译 SASS 或 Less 的更多信息。

### JavaScript

Laravel 不要求你使用特定的 JavaScript 框架或库来构建应用程序。实际上，你根本不必使用JavaScript。但是，Laravel 确实包含了一些基本的脚手架，以便更容易开始使用 [Vue](https://vuejs.org/) 库编写现代 JavaScript。 Vue 提供了一个富有表现力的 API，用于使用组件构建健壮的 JavaScript应用程序。与 CSS 一样，我们可以使用 Laravel Mix 轻松地将 JavaScript 组件编译为单个浏览器准备就绪的 JavaScript 文件。

### 移除前端脚手架

如果要从应用程序中删除前端脚手架，可以使用 `preset` Artisan 命令。此命令与 `none` 选项结合使用时，将从应用程序中删除 Bootstrap 和 Vue 脚手架，只留下一个空白的 SASS 文件和一些常用的 JavaScript 实用程序库：

```bash
php artisan preset none
```

## 编写 CSS

Laravel 的 `package.json` 文件包含 `bootstrap` 包以辅助你开始使用 Bootstrap 构建你的应用程序前端原型。然而，根据你的应用程序的需要从 `package.json` 文件中自由的添加和移除包。你不需要使用 Bootstrap 框架去构建你的 Laravel 应用程序——对于选择使用它的人来说，它提供了一个好的起点。

编译你的 CSS 之前，使用 [Node 包管理器（NPM）](https://www.npmjs.com/) 安装你的项目前端依赖项：

```bash
npm install
```

一旦使用 `npm install` 安装依赖项之后，你可以使用 [Laravel Mix](https://laravel.com/docs/5.8/mix#working-with-stylesheets) 编译你的 SASS 文件为纯 CSS。`npm run dev` 命令将处理 `webpack.mix.js` 文件中的指令。通常，你编译的 CSS 将位于 `public/css` 目录中：

```bash
npm run dev
```

Laravel 包含的默认 `webpack.mix.js` 将编译 `resources/sass/app.scss` SASS 文件。这个 `app.scss` 文件导入一个 SASS 变量文件并加载 Bootstrap，这为大多数应用程序提供了一个很好的起点。无论你希望通过 [配置 Laravel Mix](https://laravel.com/docs/5.8/mix) 随意自定义 `app.scss` 文件，甚至可以使用完全不同的预处理器。

## 编写 JavaScript

你可以在项目根目录下的 `package.json` 文件中找到应用程序所需的所有 JavaScript 依赖项。此文件类似于 `composer.json` 文件，除了它指定 JavaScript 依赖项而不是 PHP 依赖项外。你可以使用 [Node 包管理器（NPM）](https://www.npmjs.com/) 安装这些依赖项：

```bash
npm install
```

{% hint style="info" %}

默认情况下，Laravel `package.json` 文件包含一些包，比如：`vue` 和 `axios` 以辅助你开始构建你的 JavaScript 应用。根据你的应用程序的需要从 `package.json` 文件中随意地添加和移除包。

{% endhint %}

安装软件包后，可以使用 `npm run dev` 命令去 [编译资产](https://laravel.com/docs/5.8/mix)。Webpack 是现代 JavaScript 应用程序的模块捆绑器。运行 `npm run dev` 命令时，Webpack将执行 `webpack.mix.js` 文件中的指令：

```bash
npm run dev
```

默认情况下，Laravel `webpack.mix.js` 文件编译你的 SASS 和 `resources/js/app.js` 文件。在 `app.js` 文件中，你可以注册你的 Vue 组件，或者，如果你偏好不同的框架，配置你自己的 JavaScript 应用程序。你编译的 JavaScript 通常会位于 `public/js` 目录中。

## 编写 Vue 组件
