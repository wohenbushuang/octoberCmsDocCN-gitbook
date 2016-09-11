# CMS 主题

- [简介](#introduction)
- [目录结构](#directory-structure)
    - [子目录](#subdirectories)
- [模板结构](#template-structure)
    - [配置部分](#configuration-section)
    - [PHP 代码部分](#php-section)
    - [Twig markup部分](#twig-section)

<a name="introduction"></a>
## 简介

主题定义了你用 October 建立网站或 web 程序的外观。 October 主题全部基于文件，并且可以通过任何版本系统管理，比如说 Git 。这个页面给你 October 主题的高层次描述。你可以在[页面](pages)、[局部](partials)、[布局](layouts)和[内容](content)相应文章找到更多细节。

主题是默认在文件夹 **/themes** 中的文件夹。主题可以包含以下对象：

对象 | 描述
------------- | -------------
[页面](pages) | 展现网页页面。
[局部](partials) | 包含可复用 HTML markup 块。
[布局](layouts) | 定义页面脚手架。
[内容](content) | 可与页面或布局分离单独编辑的文本、 HTML 或 [Markdown](http://daringfireball.net/projects/markdown/syntax) 块。
**Asset 文件** | 诸如图像、 CSS 和 JavaScript 的资源文件。

<a name="directory-structure"></a>
## 目录结构

以下你可以看到一个示例主题的文件夹结构。每个 October 使用分开的目录，一般来说将有一个活动的主题用来展示网站。这个示例将展示的是 "website" 主题文件夹。

    themes/
      website/           <=== 主题从此开始
        pages/           <=== 页面目录
          home.htm
        layouts/         <=== 布局目录
          default.htm
        partials/        <=== 局部目录
          sidebar.htm
        content/         <=== 内容目录
          intro.htm
        assets/          <=== Assets 目录
          css/
            my-styles.css
          js/
          images/

> 活动主题可以通过文件 `config/cms.php` 中的 `activeTheme` 参数或者后端页中系统>CMS>前端主题的主题选择器来设定。主题选择器的主题设定将覆写文件 `config/cms.php` 中的值。

<a name="subdirectories"></a>
### 子目录

October 支持页面、局部、布局和内容文件的单层级子目录（ **assets** 目录可以有任意结构）。这简化了大网站的组织。在下面示例的目录结构中你可以看到页面和局部目录包含了 **blog** 子目录，内容目录包含了 **home** 子目录。

    themes/
      website/
        pages/
          home.htm
          blog/                  <=== 子目录
            archive.htm
            category.htm
        partials/
          sidebar.htm
          blog/                  <=== 子目录
            category-list.htm
        content/
          footer-contacts.txt
          home/                  <=== 子目录
            intro.htm
        ...

从子目录引用局部或内容文件时，在模板名前指明子目录名。渲染子目录中局部的范例：

    {% partial "blog/category-list" %}

> **提示:** 模板永远都是绝对路径。如果在局部中渲染同子文件夹的另一个局部，你仍需要指明子目录名。

<a name="template-structure"></a>
## 模板结构

页面、局部和布局模板可以至少包括3个部分： **配置**、**PHP 代码**和 **Twig markup**。
部分间可以可以用`==`序列分开。
举个例子：

    url = "/blog"
    layout = "default"
    ==
    function onStart()
    {
        $this['posts'] = ...;
    }
    ==
    <h3>Blog archive</h3>
    {% for post in posts %}
        <h4>{{ post.title }}</h4>
        {{ post.content }}
    {% endfor %}

<a name="configuration-section"></a>
### 配置部分

配置部分设置了模板的参数。相应文档文章中明确并描述了不同的CMS模板支持的配置参数。配置部分使用简单的 [INI 格式](http://en.wikipedia.org/wiki/INI_file)，字符串参数值用引号包括。页面模板的配置部分示例：

    url = "/blog"
    layout = "default"

    [component]
    parameter = "value"

<a name="php-section"></a>
### PHP 代码部分

PHP 部分中的代码在每次模板渲染前执行。 PHP 部分对于所有 CMS 模板都是可选的，内容取决于定义的模板类型。 PHP 代码部分可以包含可选的 PHP 开启关闭标签来在编辑器中打开语法高亮。开启关闭标签总应当在除部分分隔外的另一行中明确。

    url = "/blog"
    layout = "default"
    ==
    <?
    function onStart()
    {
        $this['posts'] = ...;
    }
    ?>
    ==
    <h3>Blog archive</h3>
    {% for post in posts %}
        <h4>{{ post.title }}</h4>
        {{ post.content }}
    {% endfor %}

在 PHP 部分中你只能定义函数和用 PHP 的 `use` 关键字来引用命名空间。 PHP 部分不允许其他 PHP 代码。这是因为 PHP 部分在页面解析时将被转换成一个 PHP 的类。使用命名空间的范例：

    url = "/blog"
    layout = "default"
    ==
    <?
    use Acme\Blog\Classes\Post;

    function onStart()
    {
        $this['posts'] = Post::get();
    }
    ?>
    ==

设置变量的通常做法是使用数组的方法来访问 `$this` ，虽然你可以简单地使用**只读对象访问**的方法，举个例子：

    // Write via array
    $this['foo'] = 'bar';

    // Read via array
    echo $this['foo'];

    // Read-only via object
    echo $this->foo;

<a name="twig-section"></a>
### Twig markup 部分

Twig 部分定义了模板需要渲染的 markup 。在 Twig 部分中你可以使用 [October 提供](../markup)的函数、标签和过滤器，所有[原生 Twig 功能](http://twig.sensiolabs.org/documentation)，或者[插件提供的功能](../plugin/registration#extending-twig)。 Twig 部分的内容取决于模板类型（页面、布局或局部）。你可以在文档中找到具体 Twig 对象的更多信息。

[在 Markup 指南中](../markup)可以找到更多信息。
