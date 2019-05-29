# API 认证

## 简介

默认情况下，Laravel 通过将一个随机令牌分配给应用程序的每个用户以提供简单的 API 认证解决方案。 在 `config/auth.php` 配置文件中，已经定义了 `api` 守卫并使用了 `token` 驱动程序。此驱动程序负责检查传入请求上的 API 令牌，并验证它是否与数据库中分配给用户令牌匹配。

{% hint style="danger" %}

**注意**：虽然 Laravel 附带一个简单的、基于令牌的身份验证守卫，但是我们强烈建议您考虑使用 [Laravel Passport](https://laravel.com/docs/5.8/passport) 来实现提供 API 认证的健壮的生产应用程序。

{% endhint %}

## 配置

## 生成令牌

## 保护路由

## 在请求中传递令牌
