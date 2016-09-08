# 安装


- [最低系统要求](#system-requirements)
- [向导安装](#wizard-installation)
    - [安装疑难解答](#troubleshoot-installation)
- [Command-line installation](#command-line-installation)
- [Post-installation steps](#post-install-steps)
    - [Delete installation files](#delete-install-files)
    - [Review configuration](#config-review)
    - [Setting up the scheduler](#crontab-setup)
    - [Setting up queue workers](#queue-setup)

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
1. [下载安装包](http://octobercms.com/download)。
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
## Command-line installation

If you feel more comfortable with a command-line or want to use composer, there is a CLI install process on the [Console interface page](../console/commands#console-install).

<a name="post-install-steps"></a>
## Post-installation steps

There are some things you may need to set up after the installation is complete.

<a name="delete-install-files"></a>
### Delete installation files

If you have used the [Wizard installer](#wizard-installation) you should delete the installation files for security reasons. October will never delete files from your system automatically, so you should delete these files and directories manually:

    install_files/      <== Installation directory
    install.php         <== Installation script

<a name="config-review"></a>
### Review configuration

Configuration files are stored in the **config** directory of the application. While each file contains descriptions for each setting, it is important to review the [common configuration options](../setup/configuration) available for your circumstances.

For example, in production environments you may want to enable [CSRF protection](../setup/configuration#csrf-protection). While in development environments, you may want to enable [bleeding edge updates](../setup/configuration#edge-updates).

While most configuration is optional, we strongly recommend disabling [debug mode](../setup/configuration#debug-mode) for production environments.

<a name="crontab-setup"></a>
### Setting up the scheduler

For *scheduled tasks* to operate correctly, you should add the following Cron entry to your server. Editing the crontab is commonly performed with the command `crontab -e`.

    * * * * * php /path/to/artisan schedule:run >> /dev/null 2>&1

Be sure to replace **/path/to/artisan** with the absolute path to the *artisan* file in the root directory of October. This Cron will call the command scheduler every minute. Then October evaluates any scheduled tasks and runs the tasks that are due.

> **Note**: If you are adding this to `/etc/cron.d` you'll need to specify a user immediately after `* * * * *`.

<a name="queue-setup"></a>
### Setting up queue workers

You may optionally set up an external queue for processing *queued jobs*, by default these will be handled asynchronously by the platform. This behavior can be changed by setting the `default` parameter in the `config/queue.php`.

If you decide to use the `database` queue driver, it is a good idea to add a Crontab entry for the command `php artisan queue:work` to process the first available job in the queue.
