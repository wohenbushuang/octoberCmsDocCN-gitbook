# 配置

- [服务器配置](#webserver-configuration)
    - [Apache 配置](#apache-configuration)
    - [Nginx 配置](#nginx-configuration)
    - [Lighttpd 配置](#lighttpd-configuration)
    - [IIS 配置](#iis-configuration)
- [程序配置](#app-configuration)
    - [Debug 模式](#debug-mode)
    - [安全模式](#safe-mode)
    - [CSRF 防护](#csrf-protection)
    - [最新的更新](#edge-updates)
    - [环境配置](#environment-config)
- [高级配置](#advanced-configuration)
    - [使用 public 文件夹](#public-folder)
    - [拓展环境配置](#environment-config-extended)

所有 October 的配置文件都储存在 **config/** 目录下。每个配置项都在文件中，所以你可以很方便地浏览这些文件并熟悉对你有用的配置项。

<a name="webserver-configuration"></a>
## 服务器配置

October 需要在你的服务器上应用一些基础配置。以下是常用的服务器及他们的配置方法。

<a name="apache-configuration"></a>
### Apache 配置

运行 Apache 的服务器有以下额外的系统要求:

1. 需要安装 mod_rewrite 
1. 需要打开 AllowOverride 选项

某些情况下你可能需要将 `.htaccess` 文件里的这一行取消注释：

    ##
    ## You may need to uncomment the following line for some hosting environments,
    ## if you have installed to a subdirectory, enter the name here also.
    ##
    # RewriteBase /

如果你安装到了子目录，你需要同时加上子目录的名称：

    RewriteBase /mysubdirectory/

<a name="nginx-configuration"></a>
### Nginx 配置

在 Nginx 服务器中需要配置一些小更改。

`nano /etc/nginx/sites-available/default`

在 **server** 节中使用以下代码。如果你把 October 安装在子目录中，则把 `/` 替换成 October 的安装地址：

    location / {
        try_files $uri $uri/ /index.php$is_args$args;
    }

    rewrite ^themes/.*/(layouts|pages|partials)/.*.htm /index.php break;
    rewrite ^bootstrap/.* /index.php break;
    rewrite ^config/.* /index.php break;
    rewrite ^vendor/.* /index.php break;
    rewrite ^storage/cms/.* /index.php break;
    rewrite ^storage/logs/.* /index.php break;
    rewrite ^storage/framework/.* /index.php break;
    rewrite ^storage/temp/protected/.* /index.php break;
    rewrite ^storage/app/uploads/protected/.* /index.php break;

<a name="lighttpd-configuration"></a>
### Lighttpd 配置

如果你的服务器使用 Lighttpd 你可以使用以下配置来运行 OctoberCMS。使用你喜爱的编辑器打开你网站的配置文件。

`nano /etc/lighttpd/conf-enabled/sites.conf`

在编辑器中粘贴以下代码并根据你项目（的实际情况）改动 **host address** 和 **server.document-root** 。

    $HTTP["host"] =~ "example.domain.com" {
        server.document-root = "/var/www/example/"

        url.rewrite-once = (
            "^/(plugins|modules/(system|backend|cms))/(([\w-]+/)+|/|)assets/([\w-]+/)+[-\w^&'@{}[\],$=!#().%+~/ ]+\.(jpg|jpeg|gif|png|svg|swf|avi|mpg|mpeg|mp3|flv|ico|css|js|woff|ttf)(\?.*|)$" => "$0",
            "^/(system|themes/[\w-]+)/assets/([\w-]+/)+[-\w^&'@{}[\],$=!#().%+~/ ]+\.(jpg|jpeg|gif|png|svg|swf|avi|mpg|mpeg|mp3|flv|ico|css|js|woff|ttf)(\?.*|)$" => "$0",
            "^/storage/app/uploads/public/[\w-]+/.*$" => "$0",
            "^/storage/temp/public/[\w-]+/.*$" => "$0",
            "^/(favicon\.ico|robots\.txt|sitemap\.xml)$" => "$0",
            "(.*)" => "/index.php$1"
        )
    }

<a name="iis-configuration"></a>
### IIS 配置

如果你的服务器使用 Internet Information Services (IIS) 你可以在 **web.config** 中使用以下配置来运行 OctoberCMS。

    <?xml version="1.0" encoding="UTF-8"?>
    <configuration>
        <system.webServer>
            <rewrite>
                <rules>
                    <rule name="redirect all requests" stopProcessing="true">
                        <match url="^(.*)$" ignoreCase="false" />
                        <conditions logicalGrouping="MatchAll">
                            <add input="{REQUEST_FILENAME}" matchType="IsFile" negate="true" pattern="" ignoreCase="false" />
                        </conditions>
                        <action type="Rewrite" url="index.php" appendQueryString="true" />
                    </rule>
                </rules>
            </rewrite>
        </system.webServer>
    </configuration>

<a name="app-configuration"></a>
## 程序配置

<a name="debug-mode"></a>
### Debug 模式

可以在配置文件 `config/app.php` 中的 `debug` 变量找到 debug 选项，默认开启。

当这个选项开启时，（页面）将会显示其他正在开发的功能发出的详细错误信息。虽然在开发时 debug 模式很实用，在线上生产环境站点中总是应该关闭 debug 模式。这将预防潜在的敏感信息被展现给终端用户。

debug 模式打开时有以下功能特性：

1. 展示[详细错误页面](cms-pages#error-page)。
1. 为用户身份验证失败提供明确原因。
1. [整合的资源文件](markup-filter-theme)默认不压缩。
1. [安全模式](#safe-mode) 默认关闭。

> **重要**：在生产环境中切记将 `app.debug` 设为 `false` 。

<a name="safe-mode"></a>
### 安全模式

可以在配置文件 `config/cms.php` 中的 `enableSafeMode` 变量找到安全模式选项。默认值是 `null` 。

如果打开安全模式， CMS 模版中的 PHP 代码段将因安全原因关闭。如果设为 `null` ，安全模式将在 [debug 模式](#debug-mode) 关闭时启用。

<a name="csrf-protection"></a>
### CSRF 防护

October 提供简单的方法来保护你的程序远离跨站请求伪造。首先一个随机 token 将被放入你用户的 session 中。之后当[使用开放表单标签](services-html#form-tokens)时 token 将被加入页面并随每个请求被提交回来。

虽然 CSRF 防护默认关闭，你可以通过配置文件 `config/cms.php` 中的 `enableCsrfProtection` 变量打开。

<a name="edge-updates"></a>
### 最新的更新

October 平台和一些插件会分两步实现改动以保证平台的整体稳定性和完整性。这意味着他们除默认的*稳定版*外还有*测试版*。

你可以通过改变配置文件 `config/cms.php` 中的 `edgeUpdates` 变量让平台优先选择测试版。

    /*
    |--------------------------------------------------------------------------
    | Bleeding edge updates
    |--------------------------------------------------------------------------
    |
    | If you are developing with October, it is important to have the latest
    | code base, set this value to 'true' to tell the platform to download
    | and use the development copies of core files and plugins.
    |
    */

    'edgeUpdates' => false,

> **提示:** 对插件的开发者我们建议在插件设置页面为你已列在市场中的插件启用 **Test updates** 。

<a name="environment-config"></a>
### 环境配置

一般来说基于程序运行的环境使用不同配置值很有帮助。你可以通过设置默认为 **production** 的 `APP_ENV` 环境变量来实现。有两种普通的方式来改变这个值：

1. 直接用你的服务器设置 `APP_ENV` 值。

    比如说，对于 Apache 可以向文件 `.htaccess` 或 `httpd.config` 添加这行：

        SetEnv APP_ENV "dev"

2. 在根目录新建一个 **.env** 文件，内容如下：

        APP_ENV=dev

以上两个例子中，环境被设为新的值 `dev` 。现在可以在 **config/dev** 路径内新建配置文件，并将覆盖程序的基本配置。

举个例子，为了只在 `dev` 环境中使用一个不同的 MySQL 数据库，以以下内容新建一个文件 **config/dev/database.php** 

    <?php

    return [
        'connections' => [
            'mysql' => [
                'host'     => 'localhost',
                'port'     => '',
                'database' => 'database',
                'username' => 'root',
                'password' => ''
            ]
        ]
    ];

<a name="advanced-configuration"></a>
## 高级配置

<a name="public-folder"></a>
### 使用 public 文件夹

为了生产环境中的高安全性你可以配置你的服务器使用 **public/** 文件夹来保证只有开放的文件能被访问。首先你需要使用 `october:mirror` 命令建立一个开放文件夹。

    php artisan october:mirror public/

这将在项目基目录新建一个文件夹 **public/** ，之后修改服务器配置来使用这个新路径作为主页目录，也被称作 *wwwroot*。

> **提示**：运行以上命令可能需要使用系统管理员或 *sudo* 权限。每次系统更新或新插件安装后可能都需要再次运行。

<a name="environment-config-extended"></a>
### 拓展环境配置

作为标准[环境配置](#environment-config)的备选你可能想使用一般值而不是使用配置文件。在此情况下将使用 [DotEnv](https://github.com/vlucas/phpdotenv) 语法访问配置。运行 `october:env` 命令迁移一般配置值到环境中：

    php artisan october:env

这将在项目根目录新建 **.env** 文件并修改配置文件来使用 `env` 帮助函数。第一个参数含有可在环境中找到的键名，第二个参数含有一个可选的默认值。

    'debug' => env('APP_DEBUG', true),

不应该向你程序的代码控制提交你的 `.env` 文件，因为每个开发者或者使用你程序的服务器可以使用不同的环境配置。
