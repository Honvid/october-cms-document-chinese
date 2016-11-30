所有的October的配置文件都放在`config/`目录下。每一个配置选项都有详细的文档说明，你可以非常方便的浏览和配置他们。

## Web服务器配置

October提供了基本的服务器配置，下面可以找到一些常见的Web服务器以及配置示例。

### Apache配置

如果你的服务器使用了apache，那么需要满足下面额外的系统条件：

1. 安装`mod_rewrite`模块
2. 开启`AllowOverride`

某些情况下，需要去掉`.htaccess`文件中的的注释：
```
##
## You may need to uncomment the following line for some hosting environments,
## if you have installed to a subdirectory, enter the name here also.
##
# RewriteBase /
```
如果项目安装在子目录，那么需要把子目录添加的下面的代码中：
```
RewriteBase /mysubdirectory/
```
### Nginx配置

Nginx环境下只需要做较小的改动。
```
nano /etc/nginx/sites-available/default
```
把下面的代码片段复制到`server`中。如果October安装在了子目录，那么请替换`/`为October的安装目录：

```
location / {
    try_files $uri /index.php$is_args$args;
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

```

### Lighttpd配置

如果网站使用的是Lighttpd，你可以使用下面的配置来运行October CMS。用你喜欢的编辑器打开网站配置文件。

```
nano /etc/lighttpd/conf-enabled/sites.conf
```

粘贴下面的代码，并替换`host name`和`server.document-root`为项目对应的内容。

```
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
```

### IIS配置

如果使用**Internet Information Services (IIS)**，你可以在`web.config`中使用下面的配置来运行October CMS。

```
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
```
## 应用配置

### 调试模式

调试模式的配置可以在`config/app.php`找到`debug`参数，默认值为开启。

当调试模式启用时，此设置将随着其他调试功能而显示详细的错误消息。虽然在开发过程中非常有用，但不建议在生产环境中使用，应该始终禁用调试模式，这样可以防止潜在的敏感信息被显示给最终用户。

当开启时，将使用一下功能：

1. 显示详细的错误页
2. 提供用户身份验证的错误特定原因
3. 静态资源使用默认未压缩的文件
4. 安全模式默认禁用

> 重要：生产环境中，无论任何时候`app.debug`需要设置为`false`

### 安全模式
安全模式配置可以在`config/cms.php`中的`enableSafeMode`参数中设置，默认为`null`。
安全模式启用时，因安全原因PHP代码不能在CMS模版中使用。如果设置为`null`，禁用调试模式时，安全模式是开启状态。
### CSRF保护
October 提供了一个简单的方法来保护应用不被跨站伪造请求攻击。首先在用户端随机生成一个`token`，然后当打开一个带有`Form`标签的页面时，自动将`token`添加到的这个页面，并在每次请求的时候一并提交到后台。
然而CSRF保护默认是关闭的，你可以通过`config/cms.php`文件中的`enableCsrfProtection`参数来启用。
### 最新更新
October平台和一些插件一般采用两种方式来确保整个平台的整体稳定性和完整性。这意味着他们会有一个基于稳定版的测试版本。
你可以通过配置文件`config/cms.php`中的`edgeupdates`参数的改变来让平台选择测试版本。
```
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
```
> 注意：对于插件开发人员，我们建议您在市场上列出插件的测试更新，并通过插件设置页面为您的测试更新。

### 环境配置
通常，基于应用的运行环境配置不同的配置是非常有帮助的，你可以通过设置`APP_ENV`环境变量来实现，默认为`production`。这里有两种方式：

1. 直接在服务器上设置`APP_ENV`的值。
例如在Apache环境中，添加下面代码到`.htaccess`或`httpd.config`文件中。
```
SetEnv APP_ENV "dev"
```

2. 在根目录下创建`.env`文件，填写以下内容：
```
APP_ENV=dev
```

上面的两种示例已经把环境变量设置为`dev`，现在可以在`config/dev`目录中创建配置文件，这些配置文件会覆盖应用的基本配置。
比如，仅在`dev`环境下使用不同的MySQL数据库，在`config/dev/database.php`中添加如下内容：

```
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
```
## 高级配置

### 使用`public`目录

为保障生产环境中的高度安全性，需要配置Web服务器使用`pulic/`文件夹，以确保只有公共文件可以被访问。首先，你需要用`october:mirror`命令生成一个公共文件夹。

```
php artisan october:mirror public/
```

这会在项目的根目录创建一个`public`文件夹，到这里，你需要编辑服务器的配置来使用这个新目录作为主目录，也被称为`wwwroot`。

> 注意：上面的命令执行的时候需要系统管理员或者_sudo_权限。同时，也需要在系统更新或者安装新的插件之后执行。

### 扩展环境配置

作为一种替代标准环境配置的方法，你可以在环境中放置公共配置，而不是使用配置文件。这个配置是利用[DotEnv](')语法访问。运行`october:env`命令移动共同配置到系统环境：

```
php artisan october:env
``` 

这将在项目根目录里创建一个`.env`文件，修改相关配置可以使用`env`的辅助函数。第一个参数是环境中`key`，第二个参数包含一个可选的默认值。
```
'debug' => env('APP_DEBUG', true),
```
你的`.env`文件不能提交到应用的版本控制中，因为每一位使用应用的开发者或者服务器都会有不同的环境配置。
