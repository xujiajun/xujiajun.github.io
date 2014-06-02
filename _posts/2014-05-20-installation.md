---
layout: post
title: 安装和配置SYMFONY [第3天]
date: 2014-05-20 7:39:31
disqus: y
---

The goal of this chapter is to get you up and running with a working application built on top of Symfony. Fortunately, Symfony offers "distributions", which are functional Symfony "starter" projects that you can download and begin developing in immediately.

Tips:

If you're looking for instructions on how best to create a new project and store it via source control, see Using Source Control.

Installing a Symfony2 Distribution¶

Tips:

First, check that you have installed and configured a Web server (such as Apache) with PHP. For more information on Symfony2 requirements, see the requirements reference.

Symfony2 packages "distributions", which are fully-functional applications that include the Symfony2 core libraries, a selection of useful bundles, a sensible directory structure and some default configuration. When you download a Symfony2 distribution, you're downloading a functional application skeleton that can be used immediately to begin developing your application.
Start by visiting the Symfony2 download page at http://symfony.com/download. On this page, you'll see the Symfony Standard Edition, which is the main Symfony2 distribution. There are 2 ways to get your project started:

Option 1) Composer¶
Composer is a dependency management library for PHP, which you can use to download the Symfony2 Standard Edition.
Start by downloading Composer anywhere onto your local computer. If you have curl installed, it's as easy as:

本章的目标是让你和运行与建立在Symfony的顶部工作的应用程序。幸运的是，Symfony的提供了“分布”，这是你可以下载并立即开始开发在功能Symfony的“启动器”项目。

温馨提示：

如果你正在寻找如何最好地创建一个新的项目，它通过源代码控制存储，请参阅使用源代码控制指令。

安装Symfony2的分布¶

温馨提示：

首先，检查是否已安装并配置PHP的Web服务器（例如Apache）。有关Symfony2的要求的详细信息，请参阅需求参考。

Symfony2的包“分配”，这是全功能的应用程序，其中包括Symfony2的核心库，选择有用的包，一个合理的目录结构和一些默认配置。当您下载Symfony2的分布，你下载了可以马上用来开始开发你的应用程序的功能的应用程序框架。首先访问Symfony2的下载页面http://symfony.com/download。在这个页面，你会看到Symfony的标准版，这是主要的Symfony2的分布。有2种方法让你的项目开始：

选项​​1)Composer是一个PHP依赖管理库，你可以用它来下载Symfony2的标准版。首先，在任何地方下载Composer到您的本地计算机上。如果你已经安装了curl：

{% highlight PHP %}
$ curl -s https://getcomposer.org/installer | php
{% endhighlight %}

Tips:

If your computer is not ready to use Composer, you'll see some recommendations when running this command. Follow those recommendations to get Composer working properly.

Composer is an executable PHAR file, which you can use to download the Standard Distribution:

{% highlight PHP %}
$ php composer.phar create-project symfony/framework-standard-edition /path/to/webroot/Symfony 2.4.*
{% endhighlight %}

Tips:

To download the vendor files faster, add the --prefer-dist option at the end of any Composer command.

This command may take several minutes to run as Composer downloads the Standard Distribution along with all of the vendor libraries that it needs. When it finishes, you should have a directory that looks something like this:


{% highlight PHP %}
path/to/webroot/ <- your web server directory (sometimes named htdocs or public)
    Symfony/ <- the new directory
        app/
            cache/
            config/
            logs/
        src/
            ...
        vendor/
            ...
        web/
            app.php
            ...
{% endhighlight %}

Option 2) Download an Archive¶

You can also download an archive of the Standard Edition. Here, you'll need to make two choices:
Download either a .tgz or .zip archive - both are equivalent, download whatever you're more comfortable using;

Download the distribution with or without vendors. If you're planning on using more third-party libraries or bundles and managing them via Composer, you should probably download "without vendors".
Download one of the archives somewhere under your local web server's root directory and unpack it. From a UNIX command line, this can be done with one of the following commands (replacing ### with your actual filename):


{% highlight PHP %}
# for .tgz file
$ tar zxvf Symfony_Standard_Vendors_2.4.###.tgz

# for a .zip file
$ unzip Symfony_Standard_Vendors_2.4.###.zip
{% endhighlight %}

If you've downloaded "without vendors", you'll definitely need to read the next section.

Tips:

You can easily override the default directory structure. See How to override Symfony's Default Directory Structure for more information.

All public files and the front controller that handles incoming requests in a Symfony2 application live in the Symfony/web/ directory. So, assuming you unpacked the archive into your web server's or virtual host's document root, your application's URLs will start with http://localhost/Symfony/web/.

The following examples assume you don't touch the document root settings so all URLs start with http://localhost/Symfony/web/

Updating Vendors¶

At this point, you've downloaded a fully-functional Symfony project in which you'll start to develop your own application. A Symfony project depends on a number of external libraries. These are downloaded into the vendor/ directory of your project via a library called Composer.

Depending on how you downloaded Symfony, you may or may not need to update your vendors right now. But, updating your vendors is always safe, and guarantees that you have all the vendor libraries you need.

Step 1: Get Composer (The great new PHP packaging system)

{% highlight PHP %}
$ curl -s http://getcomposer.org/installer | php
{% endhighlight %}

Make sure you download composer.phar in the same folder where the composer.json file is located (this is your Symfony project root by default).

Step 2: Install vendors

{% highlight PHP %}
$ php composer.phar install
{% endhighlight %}

This command downloads all of the necessary vendor libraries - including Symfony itself - into the vendor/ directory.

If you don't have curl installed, you can also just download the installer file manually at http://getcomposer.org/installer. Place this file into your project and then run:

{% highlight PHP %}
$ php installer
$ php composer.phar install
{% endhighlight %}


Tips:

When running php composer.phar install or php composer.phar update, Composer will execute post install/update commands to clear the cache and install assets. By default, the assets will be copied into your web directory.
Instead of copying your Symfony assets, you can create symlinks if your operating system supports it. To create symlinks, add an entry in the extra node of your composer.json file with the key symfony-assets-install and the value symlink:

{% highlight PHP %}
"extra": {
    "symfony-app-dir": "app",
    "symfony-web-dir": "web",
    "symfony-assets-install": "symlink"
}
{% endhighlight %}

When passing relative instead of symlink to symfony-assets-install, the command will generate relative symlinks.

Configuration and Setup¶

At this point, all of the needed third-party libraries now live in the vendor/ directory. You also have a default application setup in app/ and some sample code inside the src/ directory.

Symfony2 comes with a visual server configuration tester to help make sure your Web server and PHP are configured to use Symfony. Use the following URL to check your configuration:

{% highlight PHP %}
http://localhost/config.php
{% endhighlight %}

If there are any issues, correct them now before moving on.

