# JavaScript & CSS 脚手架

## 简介

虽然 Laravel 没有规定你使用哪种 JavaScript 或 CSS 预处理器，但它确实提供了使用 [Bootstrap](https://getbootstrap.com/) 和 [Vue](https://vuejs.org/) 的基本起点，这对许多应用程序都有帮助。默认情况下，Laravel 使用 [NPM](https://www.npmjs.com/) 来安装这两个前端软件包。

### CSS

[Laravel-Mix](https://laravel.com/docs/5.8/mix) 提供了一个干净，富有表现力的API，而不是编译 SASS 或 Less，这是纯 CSS 的扩展，增加了变量，mixins 和其他强大的功能，使得使用 CSS 变得更加愉快。在本文档中，我们将简要讨论 CSS 编译；但是，你应该查阅完整的 [Laravel Mix 文档](https://laravel.com/docs/5.8/mix)，以获取有关编译 SASS 或 Less 的更多信息。

### JavaScript

Laravel 不要求你使用特定的 JavaScript 框架或库来构建应用程序。实际上，你根本不必使用JavaScript。但是，Laravel 确实包含了一些基本的脚手架，以便更容易开始使用 [Vue](https://vuejs.org/) 库编写现代 JavaScript。 Vue 提供了一个富有表现力的 API，用于使用组件构建健壮的 JavaScript应用程序。与 CSS 一样，我们可以使用 Laravel Mix 轻松地将 JavaScript 组件编译为单个浏览器准备就绪的 JavaScript 文件。

### 移除前端脚手架

```bash
php artisan preset none
```
