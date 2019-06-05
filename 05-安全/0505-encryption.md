# 数据加密

## 简介

Laravel 的加密机使用 OpenSSL 提供 AES-256 和 AES-128 加密。强烈建议你使用 Laravel 内置的加密设施，而不要尝试使用你自己的『如法炮制的』加密算法。Laravel 的所有加密值都使用消息认证代码（MAC）签名，因此一旦加密，就不能修改它们的基础值。

## 配置

在使用 Laravel 的加密器之前，必须在 `config/app.php` 配置文件中设置一个 `key` 选项。你应当使用 `php artisan key:generate` 命令来生成此密钥，因为此 Artisan 命令将使用 PHP 的安全随机字节生成器来构建你的密钥。如果未正确设置此值，则 Laravel 加密的所有值都将不安全。

## 使用加密
