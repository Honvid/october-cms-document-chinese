October CMS 提供两种方式进行安装，你可以使用“安装向导程序”或者“命令行”完成安装。在安装之前，你应该确认你的服务器是否满足October的最低系统要求。

## 最低系统要求

October CMS 的服务器最低要求：

1. PHP version 5.5.9 or higher
2. PDO PHP Extension
3. cURL PHP Extension
4. OpenSSL PHP Extension
5. Mbstring PHP Library
6. ZipArchive PHP Library
7. GD PHP Library

对于PHP 5.5，有些系统可能需要手动安装PHP JSON 扩展。如果系统是Ubuntu，可以通过`apt-get install php5-json`命令进行安装。

如果使用SQL Server 数据库引擎，你需要安装[Group Concatenation](https://groupconcat.codeplex.com/) 自定义聚合插件。

## 安装向导程序

通过安装向导程序安装October CMS 是官方推荐的安装方式。它比命令行方式安装更加简单，而且不需要特殊技巧。

1. 在服务器上准备一个空目录，可以是子目录，根域名或者子域名。
2. [下载安装向导文件](http://octobercms.com/download) 。
3. 解压下载好的安装文件到之前准备好的目录。
4. 配置安装目录、子目录以及文件的可写权限。
5. 在浏览器中运行`install.php` 文件。
6. 根据安装说明完成安装。

### 安装过程可能出现的问题

1. **下载应用文件的时候出现500错误：**或许需要增加或者禁用Web服务器的超时限制。例如，Apache的FastCGI有`-idle-timeout`的选项可以设置为30秒。

2. **打开应用的时候出现空白页面：**检查文件和目录的权限是否设置正确。你可以通过运行`chmod -R 777 *`命令修复。

3. **出现错误代码“liveConnection”：**安装程序会通过80端口来尝试安装。检查Web服务器是否可以通过PHP访问80端口。联系你的主机供应商或者检查防火墙设置。

4. **MySQL出现“Syntax error or access violation:1067 Invalid default value for …”的错误：**检查并确认MySQL配置中`NO_ZERO_DATE`为不可用状态。

> 注意：可以在`install_files/install.log`中查看详细的安装日志。  

## 命令行安装

如果感觉命令行方式更亲切，或者想使用`composer`，在[控制台界面](')提供了CLI 安装进程

## 安装后的步骤

安装完成后，您可能需要完成下面的设置。 

### 删除安装文件

如果你使用了安装向导程序，安全起见，你应该删掉安装文件。October绝对不会自动删除系统上的文件，所以需要手动删除这些文件和目录：
```
install_files/      <== 安装目录
install.php         <== 安装脚本
```

### 检查配置

配置文件存放在应用的`config`目录。每一个文件中都包含配置的详细说明，检查系统环境[通用配置选项](')的可用性是非常重要的。

例如，生产环境下你可能会开启[CSRF]('')模式。开发环境下，你或许想看的实时的变化。

虽然大多数配置是可选的，但我们强烈建议在生产环境时禁用[调试模式](')

### 设置调度器

为了准确的执行计划任务，你应该添加下面Cron的内容到服务器。通常可以使用命令`crontab -e`来编辑Crontab。
```
* * * * * php /path/to/artisan schedule:run >> /dev/null 2>&1
```

请确保替换`/path/to/artisan`的路径为artisan文件的绝对路径。这个Cron将每分钟调用一次该命令。然后October会判断所有的计划任务，并且在适当的时候执行。

> 注意：如果添加`/etc/cron.d`则需要在 `* * * * *`后面添加一个指定的用户。

### 设置队列任务

你可以选择设置一个用于处理队列作业的外部队列，默认情况下，这些将由平台异步处理。 这些行为也可以通过设置`config/queue.php`文件中的参数`default`来设定。

如果你决定使用数据库队列驱动程序，那么为`php artisan queue:work`添加一个Crontab任务来处理队列中的第一条可用任务不失为一个好主意。
