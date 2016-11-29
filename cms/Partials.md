* [Introduction](#user-content-introduction)
* [Rendering partials](#user-content-rendering-partials)
* [Passing variables to partials](#user-content-passing-variables-to-partials)
* [Dynamic partials](#user-content-dynamic-partials)
    - [Partial execution life cycle](#user-content-partial-execution-life-cycle)
    - [Life cycle limitations](#user-content-life-cycle-limitations)

## Introduction

Partials contain reusable chunks of Twig markup that can be used anywhere throughout the website. Partials are extremely useful for page elements that repeat on different pages or layouts. A good partial example is a page footer which is used in different [page layouts](). Also, partials are required for [updating the page content with AJAX]().

Partial templates files reside in the **/partials** subdirectory of a theme directory. Partial files should have the **htm** extension. Example of a simplest possible partial:

```
<p>This is a partial</p>
```

The [Configuration]() section is optional for partials and can contain the optional **description** parameter which is displayed in the back-end user interface. The next example shows a partial with description:

```
description = "Demo partial"
==
<p>This is a partial</p>
```

The partial configuration section can also contain component definitions. [Components]() are explained in another article.

## Rendering partials

The `{% partial "partial-name" %}` Twig tag renders a partial. The tag has a single required parameter - the partial file name without the extension. Remember that if you refer a partial from a [subdirectory]() you should specify the subdirectory name. The `{% partial %}` tag can be used inside a page, layout or another partial. Example of a page referring to a partial:

```
<div class="sidebar">
    {% partial "sidebar-contacts" %}
</div>
```

## Passing variables to partials

You will find that you often need to pass variables to a partial from the external code. This makes partials even more useful. For example, you can have a partial that renders a list of blog post. If you can pass the post collection to the partial, the same partial could be used on the blog archive page, on the blog category page and so on. You can pass variables to partials by specifying them after the partial name in the {% partial %} tag:

```
<div class="sidebar">
    {% partial "blog-posts" posts=posts %}
</div>
```

You can also assign new variables for use in the partial:

```
<div class="sidebar">
    {% partial "sidebar-contacts" city="Vancouver" country="Canada" %}
</div>
```

Inside the partial, variables can be accessed like any other markup variable:

```
<p>Country: {{ country }}, city: {{ city }}.</p>
```

## Dynamic partials

Partials, like pages, can use any Twig features. Please refer to the [Dynamic pages]() documentation for details.

### Partial execution life cycle

There are special functions that can be defined in the PHP section of partials: `onStart()` and `onEnd()`. The `onStart()` function is executed before the partial is rendered and before the partial components are executed. The `onEnd()` function is executed before the partial is rendered and after the partial components are executed. In the onStart and onEnd functions you can inject variables to the Twig environment. Use the array notation to pass variables to the page:

```
==
function onStart()
{
    $this['hello'] = "Hello world!";
}
==
<h3>{{ hello }}</h3>
```

The templating language provided by October is described in the [Markup Guide](). The overall sequence the handlers are executed is described in the [Dynamic layouts]() article.

### Life cycle limitations

Since they are instantiated late, during the time the page is rendered, some limitations apply to the life cycle of partials. They do not follow the standard execution process, as described in the [layout execution life cycle](). The following limitations should be noted:

1. AJAX events are not registered and won't function as normal.
2. The life cycle functions cannot return any values.
3. Regular POST form handling will occur at the time the partial is rendered.

In general, component usage in partials is designed for basic components that render simple markup without much processing, such as a Like or Tweet button.
