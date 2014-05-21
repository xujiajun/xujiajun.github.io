---
layout: post
title: Installing and Configuring Symfony [第3天]
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



