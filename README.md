# 简介


> 以下简介摘自October CMS项目的 ```README.md``` ，项目地址 [https://github.com/octobercms/october](https://github.com/octobercms/october)
> 
> 译者非语言系，使用业余时间翻译，以便自己开发速查。如有翻译错误或不通顺的地方请多海涵。

October CMS，即十月内容管理系统，是一款基于Laravel开发的CMS系统，

![October](https://github.com/octobercms/october/blob/master/themes/demo/assets/images/october.png?raw=true)

[October](http://octobercms.com) 是一个内容管理系统 (CMS) ，一个让你的开发流程更加简单为唯一目的的网络平台。它的诞生源于对现有系统的失望。我们觉得建设网站已成为使开发人员费解、混淆和不满意的过程。我们希望能把你带回简单的一面，回到基础中。

October的任务是向世界展现：web开发并非是造火箭的科学（那么难）。

[![Build Status](https://travis-ci.org/octobercms/october.svg?branch=develop)](https://travis-ci.org/octobercms/october)
[![License](https://poser.pugx.org/october/october/license.svg)](https://packagist.org/packages/october/october)

### 学习 October

学习 October 最好的方法是  [阅读文档](http://octobercms.com/docs) 或者 [跟随教程](http://octobercms.com/support/articles/tutorials)。

你也可以看看这些为 [初学者](https://vimeo.com/79963873) 和 [高级用户](https://vimeo.com/172202661) 准备的介绍视频。

### 安装 October

如何安装 October 的步骤请参考 [安装指南](http://octobercms.com/docs/setup/installation).

### 快速安装

对于高级用户，在你的命令行中运行以下命令来安装 October ：

```shell
php -r "eval('?>'.file_get_contents('https://octobercms.com/api/installer'));"
```

如果你打算使用数据库，运行以下命令：

```shell
php artisan october:install
```

### 开发团队

October 由 [Alexey Bobkov](http://ca.linkedin.com/pub/aleksey-bobkov/2b/ba0/232) 和 [Samuel Georges](http://au.linkedin.com/pub/sam-georges/31/641/a9) 创造, 二人将继续开发这个平台。

### 基础库

本 CMS 使用 [Laravel](http://laravel.com) 作为 PHP 基础框架

### 联系方式

You can communicate with us using the following mediums:

* [Follow us on Twitter](http://twitter.com/octobercms) for announcements and updates.
* [Follow us on Facebook](http://facebook.com/octobercms) for announcements and updates.
* [Join us on IRC](http://octobercms.com/chat) to chat with us.

### 许可

本 OctoberCMS 平台遵循 [MIT license](http://opensource.org/licenses/MIT) 下的开源软件许可。

### 贡献

在发起 Pull 请求前，请先依照 [贡献指南](CONTRIBUTING.md) 复查。

### 编码规范

请遵循以下编程标准及指南:

* [PSR 4 Coding Standards](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-4-autoloader.md)
* [PSR 2 Coding Style Guide](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-2-coding-style-guide.md)
* [PSR 1 Coding Standards](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-1-basic-coding-standard.md)
