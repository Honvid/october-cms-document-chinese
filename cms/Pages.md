* [Introduction](#user-content-introduction)
* [Page configuration](#user-content-page-configuration)
    - [URL syntax](#user-content-url-syntax)
* [Dynamic pages](#user-content-dynamic-pages)
    - [Page execution life cycle](#user-content-page-execution-life-cycle)
    - [Sending a custom response](#user-content-sending-a-custom-response)
    - [Handling forms](#user-content-handling-forms)
* [404 page](#user-content-404-page)
* [Error page](#user-content-error-page)
* [Page variables](#user-content-page-variables)
* [Injecting page assets programmatically](#user-content-injecting-page-assets-programmatically)

## Introduction

All websites have pages. In October pages are represented with page templates. Page template files reside in the **/pages** subdirectory of a theme directory. Page file names do not affect the routing, but it's a good idea to name your pages accordingly to the page function. The files should have the **htm** extension. The [Configuration]() and [Twig]() template sections are required for pages but the [PHP section]() is optional. Below you can see the simplest home page example.

```
url = "/"
==
<h1>Hello, world!</h1>
```

## Page configuration

Page configuration is defined in the [Configuration Section]() of the page template file. The page configuration defines the page parameters, required for the routing and rendering the page and page [Components](), which are explained in another article. The following configuration parameters are supported for pages:

|Parameter|   Description|
|----|----|
|url| the page URL, required. The URL syntax is described below.|
|title|   the page title, required.|
|layout|  the page layout, optional. If specified, should contain the name of the layout file, without extension, for example: default.|
|description| the page description for the back-end interface, optional.|

### URL syntax

The page URL is defined with the **url** configuration parameter. URLs should start with the forward slash character and can contain parameters. URLs without parameters are fixed and strict. In the following example the page URL is `/blog`.

```
url = "/blog"
```

URLs with parameters are more flexible. A page with the URL pattern defined in the following example would be displayed for any address like `/blog/post/something`. URL parameters can be accessed by October components or from the page [PHP code]() section.

```
url = "/blog/post/:post_id"
```

This is how you can access the URL parameter from the page PHP section (see the [Dynamic pages]() section for more details):

```
url = "/blog/post/:post_id"
==
function onStart()
{
    $post_id = $this->param('post_id');
}
==
```

Parameter names should be compatible with PHP variable names. To make a parameter optional add the question mark after its name:

```
url = "/blog/post/:post_id?"
```

Parameters in the middle of the URL cannot be optional. In the next example the `:post_id` parameter is marked as optional, but is processed as required.

```
url = "/blog/:post_id?/comments"
```

Optional parameters can have default values which are used as fallback values in case the real parameter value is not presented in the URL. Default values cannot contain any asterisks, pipe symbols or question marks. The default value is specified after the **question mark**. In the next example the `category_id` parameter would be 10 for the URL `/blog/category`.

```
url = "/blog/category/:category_id?10"
```

You can also use regular expressions to validate parameters. To add a validation expression, add the pipe symbol after the parameter name (or the question mark) and specify the expression. The forward slash symbol is not allowed in the expressions. Examples:

```
url = "/blog/:post_id|^[0-9]+$/comments" - this will match /blog/post/10/comments
...
url = "/blog/:post_id|^[0-9]+$" - this will match /blog/post/3
...
url = "/blog/:post_name?|^[a-z0-9\-]+$" - this will match /blog/my-blog-post
```

It is possible to use a special wildcard parameter by placing an **asterisk** after the parameter. Unlike regular parameters, wildcard parameters can match one or more URL segments. A URL can only ever contain a single wildcard parameter, cannot use regular expressions, or be followed by an optional parameter.

```
url = "/blog/:category*/:slug"
```

For example, a URL like `/color/:color/make/:make*/edit` will match `/color/brown/make/volkswagen/beetle/retro/edit` and extract the following parameter values:

* color: brown
* make: volkswagen/beetle/retro

>Note: Subdirectories do not affect page URLs - the URL is defined only with the url parameter.

## Dynamic pages

Inside the [Twig section]() of a page template you can use any [functions, filters and tags provided by October](). Any dynamic page requires variables. In October page variables can be prepared by the page or layout [PHP section]() or by [Components](). In this article we describe how to prepare variables in the PHP section.

### Page execution life cycle

There are special functions that can be defined in the PHP section of pages and layouts: `onInit()`, `onStart()` and `onEnd()`. The `onInit()` function is executed when all components are initialized and before AJAX requests are handled. The `onStart()` function is executed in the beginning of the page execution. The `onEnd()` function is executed before the page is rendered and after the page components are executed. In the onStart and onEnd functions you can inject variables to the Twig environment. Use the array notation to pass variables to the page:

```
url = "/"
==
function onStart()
{
    $this['hello'] = "Hello world!";
}
==
<h3>{{ hello }}</h3>
```

The next example is more complicated. It shows how to load a blog post collection from the database and display on the page (the Acme\Blog plugin is imaginary).

```
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
```

The default variables and Twig extensions provided by October are described in the [Markup Guide](). The overall sequence the handlers are executed is described in the [Dynamic layouts]() article.

### Sending a custom response

All methods defined in the execution life cycle have the ability to halt the process and return a response. Simply return a response from the life cycle function. The example below will not load any page contents and return the string Hello world! to the browser:

```
function onStart()
{
    return 'Hello world!';
}
```

A more useful example might be triggering a redirect using the Redirect facade:

```
public function onStart()
{
    return Redirect:to('http://google.com');
}
```

### Handling forms

You can handle standard forms with handler methods defined in the page or layout [PHP section]() (handling the AJAX requests is explained in the [AJAX Framework]() article). Use the [form_open()]() function to define a form that refers to an event handler. Example:

```
{{ form_open({ request: 'onHandleForm' }) }}
    Please enter a string: <input type="text" name="value"/>
    <input type="submit" value="Submit me!"/>
{{ form_close() }}
<p>Last submitted value: {{ lastValue }}</p>
```

The onHandleForm function can be defined in the page or layout [PHP section]() in the following way:

```
function onHandleForm()
{
    $this['lastValue'] = post('value');
}
```

The handler loads the value with the `post()` function and initializes the page `lastValue` attribute variable which is displayed below the form in the first example.

>Note: If a handler with a same name is defined in the page layout, page and a page component October will execute the page handler. If a handler is defined in a component and a layout, the layout handler will be executed. The handler precedence is: page, layout, component.

If you want to refer to a handler defined in a specific component, use the component name or alias in the handler reference:

```
{{ form_open({ request: 'myComponent::onHandleForm' }) }}
```

## 404 page

If the theme contains a page with the URL /404 it is displayed when the system can't find a requested page.

## Error page

By default any errors will be shown with a detailed error page containing the file contents, line number and stack trace where the error occurred. You can display a custom error page by setting the configuration value debug to **false** in the `config/app.php` script and creating a page with the URL `/error`.

## Page variables

The properties of a page can be accessed in the [PHP code section]() or [Components]() by referencing `$this->page`.

```
function onEnd()
{
    $this->page->title = 'A different page title';
}
```

They can also be accessed in the markup using the [`this.page` variable](). For example, to return the title of a page:

```
<p>The title of this page is: {{ this.page.title }}</p>
```

More information can be found on [`this.page` in the Markup guide]().

## Injecting page assets programmatically

If needed, you can inject assets (CSS and JavaScript files) to pages with the controller's `addCss()` and `addJs()` methods. It could be done in the `onStart()` function defined in the [PHP section]() of a page or [layout]() template. Example:

```
function onStart()
{
    $this->addCss('assets/css/hello.css');
    $this->addJs('assets/js/app.js');
}
```

If the path specified in the `addCss()` and `addJs()` method argument begins with a slash (/) then it will be relative to the website root. If the asset path does not begin with a slash then it is relative to the theme.

In order to output the injected assets on pages or [layouts]() use the [{% styles %}]() and [{% scripts %}]() tags. Example:

```
<head>
    ...
    {% styles %}
</head>
<body>
    ...
    {% scripts %}
</body>
```
