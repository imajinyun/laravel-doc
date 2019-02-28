# Valet

* [简介](#jian-jie)
* [安装](#an-zhuang)
* [服务站点](#fu-wu-zhan-dian)
* [共享站点](#gong-xiang-zhan-dian)
* [自定义 Valet 驱动](#zi-ding-yi-valet-qu-dong)
* [其它 Valet 命令](#qi-ta-valet-ming-ling)

## 简介

Valet 是适用于 Mac 极简主义者的 Laravel 开发环境。没有 Vagrant，没有 `/etc/hosts` 文件。你甚至可以使用本地通道公开共享你的站点。是的，我们也喜欢它。

Laravel Valet 配置你的 Mac 在机器启动时总是在后台运行 [Nginx](https://www.nginx.com/)。然后，使用 [DnsMasq](https://en.wikipedia.org/wiki/Dnsmasq)，Valet 代理将 `*.test` 域上的所有请求指向到你的本地机器上安装的站点。

换句话说，一个超快的 Laravel 开发环境，使用了大约 7 MB 的 RAM。Valet 不是 Vagrant 或 Homestead 的完成替代品，但如果你想要灵活的基础，偏好极快的速度，或在一个受限数量的 RAM 机器上工作时，它提供了一个很好的选择。

开箱即用的 Valet 支持的，包含并不仅限于：

* [Laravel](https://laravel.com/)
* [Lumen](https://lumen.laravel.com/)
* [Bedrock](https://roots.io/bedrock/)
* [CakePHP 3](https://cakephp.org/)
* [Concrete5](https://www.concrete5.org/)
* [Contao](https://contao.org/en/)
* [Craft](https://craftcms.com/)
* [Drupal](https://www.drupal.org/)
* [Jigsaw](https://jigsaw.tighten.co/)
* [Joomla](https://www.joomla.org/)
* [Katana](https://github.com/themsaid/katana)
* [Kirby](https://getkirby.com/)
* [Magento](https://magento.com/)
* [OctoberCMS](https://octobercms.com/)
* [Sculpin](https://sculpin.io/)
* [Slim](https://www.slimframework.com/)
* [Statamic](https://statamic.com/)
* Static HTML
* [Symfony](https://symfony.com/)
* [WordPress](https://wordpress.org/)
* [Zend](https://framework.zend.com/)

不管怎样，你可以合适你自己 [自定义的驱动程序](https://laravel.com/docs/5.8/valet#custom-valet-drivers) 扩展 Valet。

## 安装

## 服务站点

## 共享站点

## 自定义 Valet 驱动

## 其它 Valet 命令
