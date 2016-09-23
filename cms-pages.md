# CMS 页面

- [简介](#introduction)
- [页面配置](#configuration)
    - [URL 语法](#url-syntax)
- [动态页面](#dynamic-pages)
    - [页面执行生命周期](#page-life-cycle)
    - [发送自定义响应](#life-cycle-response)
    - [表单处理](#handling-forms)
- [404 页面](#404-page)
- [错误页面](#error-page)
- [页面变量](#page-variables)
- [以编程方式注入页面 assets](#injecting-assets)

<a name="introduction"></a>
## 简介

所有网站都有页面。在 October ，页面使用页面模板表现。页面模板文件存在主题目录的字文件夹 **/pages** 中。页面文件名不影响路由，但根据页面功能来命名你的页面是个好主意。文件需要有 **htm** 扩展名。对于页面，[配置](themes#configuration-section)和 [Twig](themes#twig-section) 模板部分是必须的，但 [PHP 部分](themes#php-section)是可选的。下面你可以看到最简单的主页页面示例。

    url = "/"
    ==
    <h1>Hello, world!</h1>

<a name="configuration"></a>
## 页面配置

页面配置定义在页面模板文件的[配置部分](themes#configuration-section)。页面配置定义了页面变量，在路由和渲染页面和页面的[组件](components)时必需，这将在另一篇文章中解释。页面支持以下配置变量：

变量 | 描述
------------- | -------------
**url** | 页面 URL，必须。 下方解释了 URL 语法。
**title** | 页面标题，必需。
**layout** | 页面[布局](layouts)，可选。如果指明，则需包含布局文件的名字，不含扩展名，比如： `default` 。
**description** | 后台界面中的页面描述，可选。

<a name="url-syntax"></a>
### URL 语法

页面 URL 由 **url** 配置变量定义。 URL 需要以斜杠开始，且可以包含变量。不含变量的 URL 是固定严苛的。在接下来的例子中页面 URL 是 `/blog`。

    url = "/blog"

有变量的 URL 更加灵活。任意像 `/blog/post/something` 的地址将展示拥有下方例子定义的 URL 模式的页面。可以通过 October 组件或者页面 [PHP 代码](themes#php-section)部分访问 URL 变量。

    url = "/blog/post/:post_id"

这是如何从页面 PHP 部分访问 URL 变量的方法（查看[动态页面](#dynamic-pages)部分寻求更多细节）：

    url = "/blog/post/:post_id"
    ==
    function onStart()
    {
        $post_id = $this->param('post_id');
    }
    ==

变量名需要与 PHP 变量名相兼容。在变量名后加问号可以令变量可选：

    url = "/blog/post/:post_id?"

 URL 中间的变量不能是可选的。下一个例子中 `:post_id` 变量被标记为可选的，但它依必需来处理

    url = "/blog/:post_id?/comments"

可选变量可以有默认值作为退路以防 URL 中没有出现真实变量值。默认值不能带有任何星号、管道符和问号。默认值在**问号**后指明。下一个例子中，对于 URL `/blog/category` ， `category_id` 变量将是 `10` .

    url = "/blog/category/:category_id?10"

You can also use regular expressions to validate parameters. To add a validation expression, add the pipe symbol after the parameter name (or the question mark) and specify the expression. The forward slash symbol is not allowed in the expressions. Examples:

    url = "/blog/:post_id|^[0-9]+$/comments" - this will match /blog/post/10/comments
    ...
    url = "/blog/:post_id|^[0-9]+$" - this will match /blog/post/3
    ...
    url = "/blog/:post_name?|^[a-z0-9\-]+$" - this will match /blog/my-blog-post

It is possible to use a special *wildcard* parameter by placing an **asterisk** after the parameter. Unlike regular parameters, wildcard parameters can match one or more URL segments. A URL can only ever contain a single wildcard parameter, cannot use regular expressions, or be followed by an optional parameter.

    url = "/blog/:category*/:slug"

For example, a URL like `/color/:color/make/:make*/edit` will match `/color/brown/make/volkswagen/beetle/retro/edit` and extract the following parameter values:

<div class="content-list" markdown="1">
- color: `brown`
- make: `volkswagen/beetle/retro`
</div>

> **Note:** Subdirectories do not affect page URLs - the URL is defined only with the **url** parameter.

<a name="dynamic-pages"></a>
## Dynamic pages

Inside the [Twig section](themes#twig-section) of a page template you can use any [functions, filters and tags provided by October](../markup). Any dynamic page requires **variables**. In October page variables can be prepared by the page or layout [PHP section](themes#php-section) or by [Components](components). In this article we describe how to prepare variables in the PHP section.

<a name="page-life-cycle"></a>
### Page execution life cycle

There are special functions that can be defined in the PHP section of pages and layouts: `onInit()`, `onStart()` and `onEnd()`. The `onInit()` function is executed when all components are initialized and before AJAX requests are handled. The `onStart()` function is executed in the beginning of the page execution. The `onEnd()` function is executed before the page is rendered and after the page [components](components) are executed. In the onStart and onEnd functions you can inject variables to the Twig environment. Use the `array notation` to pass variables to the page:

    url = "/"
    ==
    function onStart()
    {
        $this['hello'] = "Hello world!";
    }
    ==
    <h3>{{ hello }}</h3>

The next example is more complicated. It shows how to load a blog post collection from the database and display on the page (the Acme\Blog plugin is imaginary).

    url = "/blog"
    ==
    use Acme\Blog\Classes\Post;

    function onStart()
    {
      $this['posts'] = Post::orderBy('created_at', 'desc')->get();
    }
    ==
    <h2>Latest posts</h2>
    <ul>
        {% for post in posts %}
            <h3>{{ post.title }}</h3>
            {{ post.content }}
        {% endfor %}
    </ul>

The default variables and Twig extensions provided by October are described in the [Markup Guide](../markup). The overall sequence the handlers are executed is described in the [Dynamic layouts](layouts#dynamic-layouts) article.

<a name="life-cycle-response"></a>
### Sending a custom response

All methods defined in the execution life cycle have the ability to halt the process and return a response. Simply return a response from the life cycle function. The example below will not load any page contents and return the string *Hello world!* to the browser:

    function onStart()
    {
        return 'Hello world!';
    }

A more useful example might be triggering a redirect using the `Redirect` facade:

    public function onStart()
    {
        return Redirect:to('http://google.com');
    }

<a name="handling-forms"></a>
### Handling forms

You can handle standard forms with handler methods defined in the page or layout [PHP section](themes#php-section) (handling the AJAX requests is explained in the [AJAX Framework](ajax) article). Use the [form_open()](markup#standard-form) function to define a form that refers to an event handler. Example:

    {{ form_open({ request: 'onHandleForm' }) }}
        Please enter a string: <input type="text" name="value"/>
        <input type="submit" value="Submit me!"/>
    {{ form_close() }}
    <p>Last submitted value: {{ lastValue }}</p>

The onHandleForm function can be defined in the page or layout [PHP section](themes#php-section) in the following way:

    function onHandleForm()
    {
        $this['lastValue'] = post('value');
    }

The handler loads the value with the `post()` function and initializes the page `lastValue` attribute variable which is displayed below the form in the first example.

> **Note:** If a handler with a same name is defined in the page layout, page and a page [component](components) October will execute the page handler. If a handler is defined in a component and a layout, the layout handler will be executed. The handler precedence is: page, layout, component.

If you want to refer to a handler defined in a specific [component](components), use the component name or alias in the handler reference:

    {{ form_open({ request: 'myComponent::onHandleForm' }) }}

<a name="404-page"></a>
## 404 page

If the theme contains a page with the URL `/404` it is displayed when the system can't find a requested page.

<a name="error-page"></a>
## Error page

By default any errors will be shown with a detailed error page containing the file contents, line number and stack trace where the error occurred. You can display a custom error page by setting the configuration value `debug` to **false** in the `config/app.php` script and creating a page with the URL `/error`.

<a name="page-variables"></a>
## Page variables

The properties of a page can be accessed in the [PHP code section](../cms/themes#php-section) or [Components](../cms/components) by referencing `$this->page`.

    function onEnd()
    {
        $this->page->title = 'A different page title';
    }

They can also be accessed in the markup using the [`this.page` variable](../markup/this-page). For example, to return the title of a page:

    <p>The title of this page is: {{ this.page.title }}</p>

More information can be found on [`this.page` in the Markup guide](../markup/this-page).

<a name="injecting-assets"></a>
## Injecting page assets programmatically

If needed, you can inject assets (CSS and JavaScript files) to pages with the controller's `addCss()` and `addJs()` methods. It could be done in the `onStart()` function defined in the [PHP section](themes#php-section) of a page or [layout](layout) template. Example:

    function onStart()
    {
        $this->addCss('assets/css/hello.css');
        $this->addJs('assets/js/app.js');
    }

If the path specified in the `addCss()` and `addJs()` method argument begins with a slash (/) then it will be relative to the website root. If the asset path does not begin with a slash then it is relative to the theme.

In order to output the injected assets on pages or [layouts](layout) use the [{ % styles % }](../markup/tag-styles) and [{ % scripts % }](../markup/tag-scripts) tags. Example:

    <head>
        ...
        {% styles %}
    </head>
    <body>
        ...
        {% scripts %}
    </body>
