# 安装


- [最低系统要求](#system-requirements)
- [向导安装](#wizard-installation)
    - [安装疑难解答](#troubleshoot-installation)
- [命令行安装](#command-line-installation)
- [安装后步骤](#post-install-steps)
    - [删除安装文件](#delete-install-files)
    - [复查配置](#config-review)
    - [建立定时任务](#crontab-setup)
    - [建立队列](#queue-setup)

你可以通过两种方式安装 October, [向导安装](#wizard-installation) 和 [命令行安装](../console/commands#console-install) 。 在安装前，你需要确认服务器满足最低系统要求。

<a name="system-requirements"></a>
## 最低系统要求

October CMS 对 web 服务器主机有要求：

1. PHP 版本 5.5.9 或以上
1. 已配置 PDO PHP 拓展
1. 已配置 cURL PHP 拓展
1. 已配置 OpenSSL PHP 拓展
1. 已配置 Mbstring PHP 库
1. 已配置 ZipArchive PHP 库
1. 已配置 GD PHP 库

对于 PHP 5.5, 部分分发版本系统可能要求你手动安装 PHP JSON 拓展。当你使用 Ubuntu 时，可以通过 `apt-get install php5-json` 命令完成。

当使用 SQL Server 数据库引擎，你需要安装 [group concatenation](https://groupconcat.codeplex.com/) 用户定义集合.

<a name="wizard-installation"></a>
## 向导安装

安装 October 的推荐方式是使用向导安装。这比命令行安装简单，并且不需要特殊技能。

1. 在你的服务器上准备好空文件夹。它可以是子文件夹、域名根目录或子域名目录。
1. [下载安装包](http://octobercms.com/download) 。
1. 解压缩安装包到准备的文件夹中。
1. 给予安装目录及所有子目录和文件可写权限。
1. 在你的 web 浏览器中打开 install.php 。
1. 依照安装指示进行。

![image](https://github.com/octobercms/docs/blob/master/images/wizard-installer.png?raw=true) 

<a name="troubleshoot-installation"></a>
### 安装疑难解答

1. **下载程序文件时显示500错误**：你可能需要增加或减少你 web 服务器的超时限制。比如说， Apache 的 FastCGI 的 `-idle-timeout` 选项有时候被设置为 30 秒。

1. **打开应用程序时屏幕无显示**: 确认文件和文件夹的设置正确。比如说，运行 `chmod -R 777 *` 可以修复。

1. **显示 "liveConnection" 错误码**: 安装程序会使用 80 端口测试到安装服务器的连接。检查你的 web 服务器可以通过 PHP 在 80 端口上建立传出连接。联系你的主机提供商，或者经常能在服务器防火墙中找到设置。

1. **MySQL 显示 "Syntax error or access violation: 1067 Invalid default value for ..." 错误**: 检查你的 MySQL 设置文件以确保 `NO_ZERO_DATE` 设置项已关闭。

> **提示：** 详细的安装日志可以在文件 `install_files/install.log` 中找到。

<a name="command-line-installation"></a>
## 命令行安装

如果你觉得命令行更方便，或者希望使用 composer ， 命令行界面安装过程参见 [控制台交互页面](../console/commands#console-install) 。

<a name="post-install-steps"></a>
## 安装后步骤

当安装完成后你需要做些配置。

<a name="delete-install-files"></a>
### 删除安装文件

如果你用了 [向导安装](#wizard-installation) ，为了安全原因你需要删除安装文件。 October 不会自动从你系统中删除文件，所以你需要手动删除这些文件和文件夹：

    install_files/      <== Installation directory
    install.php         <== Installation script

<a name="config-review"></a>
### 复查配置

配置文件储存在程序 **config** 目录中。虽然每个文件对每个配置项都有描述，仔细阅读对你环境可用的 [通用配置选项](../setup/configuration) 仍然很重要。

比如说，在生产环境中你可能想要打开 [CSRF 保护](../setup/configuration#csrf-protection) 。在开发环境中，你可能想要启用 [最新的更新](../setup/configuration#edge-updates) 。

虽然大部分配置项是可选的，我们强烈建议在生产环境下关闭 [debug 模式](../setup/configuration#debug-mode) 。

<a name="crontab-setup"></a>
### 建立定时任务

为了让 *定时任务* 正确运行，你需要添加下列 Cron 入口到你的服务器。编辑 crontab 通常使用命令 `crontab -e` 。

    * * * * * php /path/to/artisan schedule:run >> /dev/null 2>&1

切记将 **/path/to/artisan** 替换成 October 根目录里 *artisan* 文件的绝对地址。这个 Cron 将每分钟定时运行命令行。之后 October 将评估所有定时任务并运行到时任务。

> **提示**：如果你把 Corn 命令加入 `/etc/cron.d` ，你需要在 `* * * * *` 后指明一个（linux）用户。

<a name="queue-setup"></a>
### 建立队列

可能你希望选择配置外部队列来运行 *队列任务* ，默认上这些任务将会由这些平台异步处理。这些可以通过设置 `config/queue.php` 中的 `default` 变量来变更。

如果你打算使用 `数据库` 驱动队列，那么可以为 `php artisan queue:work` 命令添加个 Crontab 入口来运行队列中首个可用的任务。
