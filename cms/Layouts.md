* [Introduction](#user-content-introduction)
* [Placeholders](#user-content-placeholders)
* [Dynamic layouts](#user-content-dynamic-layouts)
    - [Layout execution life cycle](#user-content-layout-execution-life-cycle)

## Introduction

Layouts define the page scaffold, that is everything that repeats on a page, such as a header and footer. Layouts often contain the HTML tag as well as the HEAD, TITLE and BODY tags.

Layout templates reside in the **/layouts** subdirectory of a theme directory. Layout template files should have the **htm** extension. Inside the layout file you should use the `{% page %}` tag to output the page content. Simplest layout example:

```
<html>
    <body>
        {% page %}
    </body>
</html>
```

To use a layout for a page the page should refer the layout file name (without extension) in the Configuration section. Remember that if you refer a layout from a subdirectory you should specify the subdirectory name. Example page template using the default.htm layout:

```
url = "/"
layout = "default"
==
<p>Hello, world!</p>
```

When this page is requested its content is merged with the layout, or more precisely - the layout's `{% page %}` tag is replaced with the page content. The previous examples would generate the following markup:

```
<html>
    <body>
        <p>Hello, world!</p>
    </body>
</html>
```

Note that you can render partials in layouts. This lets you to share the common markup elements between different layouts. For example, you can have a partial that outputs the website CSS and JavaScript links. This approach simplifies the resource management - if you want to add a JavaScript reference you should modify a single partial instead of editing all the layouts.

The Configuration section is optional for layouts. The supported configuration parameters are **name** and description. The parameters are optional and used in the back-end user interface. Example layout template with a description:

```
description = "Basic layout example"
==
<html>
    <body>
        {% page %}
    </body>
</html>
```

## Placeholders

Placeholders allow pages to inject content to the layout. Placeholders are defined in the layout templates with the {% placeholder %} tag. The next example shows a layout template with a placeholder **head** defined in the HTML HEAD section.

```
<html>
    <head>
        {% placeholder head %}
    </head>
    ...
```

Pages can inject content to placeholders with the `{% put %}` and `{% endput %}` tags. The following example demonstrates a simple page template which injects a CSS link to the placeholder **head** defined in the previous example.

```
url = "/my-page"
layout = "default"
==
{% put head %}
    <link href="/themes/demo/assets/css/page.css" rel="stylesheet">
{% endput %}

<p>The page content goes here.</p>
```

More information on placeholders can be found in the Markup guide.

## Dynamic layouts

Layouts, like pages, can use any Twig features. Please refer to the Dynamic pages documentation for details.

### Layout execution life cycle

Inside the layout's PHP section you can define the following functions for handling the page execution life cycle: `onInit()`, `onStart()`, `onBeforePageStart()` and `onEnd()`.

The `onInit()` function is executed when all components are initialized and before AJAX requests are handled. The `onStart()` function is executed in the beginning of the page processing. The `onBeforePageStart()` function is executed after the layout components ran, but before the page's `onStart()` function is executed. The `onEnd()` function is executed after the page is rendered. The sequence the handlers are executed is following:

1. Layout onInit() function.
2. Page onInit() function.
3. Layout onStart() function.
4. Layout components onRun() method.
5. Layout onBeforePageStart() function.
6. Page onStart() function.
7. Page components onRun() method.
8. Page onEnd() function.
9. Layout onEnd() function.
