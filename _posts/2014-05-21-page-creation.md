---
layout: post
title: Creating Pages in Symfony2 [第4天]
date: 2014-05-21 7:39:31
disqus: y
---

Creating a new page in Symfony2 is a simple two-step process:
Create a route: A route defines the URL (e.g. /about) to your page and specifies a controller (which is a PHP function) that Symfony2 should execute when the URL of an incoming request matches the route path;

Create a controller: A controller is a PHP function that takes the incoming request and transforms it into the Symfony2 Response object that's returned to the user.
This simple approach is beautiful because it matches the way that the Web works. Every interaction on the Web is initiated by an HTTP request. The job of your application is simply to interpret the request and return the appropriate HTTP response.

Symfony2 follows this philosophy and provides you with tools and conventions to keep your application organized as it grows in users and complexity.

Environments & Front Controllers¶

Every Symfony application runs within an environment. An environment is a specific set of configuration and loaded bundles, represented by a string. The same application can be run with different configurations by running the application in different environments. Symfony2 comes with three environments defined — dev, test and prod — but you can create your own as well.
Environments are useful by allowing a single application to have a dev environment built for debugging and a production environment optimized for speed. You might also load specific bundles based on the selected environment. For example, Symfony2 comes with the WebProfilerBundle (described below), enabled only in the dev and test environments.

Symfony2 comes with two web-accessible front controllers: app_dev.php provides the dev environment, and app.php provides the prod environment. All web accesses to Symfony2 normally go through one of these front controllers. (The test environment is normally only used when running unit tests, and so doesn't have a dedicated front controller. The console tool also provides a front controller that can be used with any environment.)

When the front controller initializes the kernel, it provides two parameters: the environment, and also whether the kernel should run in debug mode. To make your application respond faster, Symfony2 maintains a cache under the app/cache/ directory. When debug mode is enabled (such as app_dev.php does by default), this cache is flushed automatically whenever you make changes to any code or configuration. When running in debug mode, Symfony2 runs slower, but your changes are reflected without having to manually clear the cache.

The "Hello Symfony!" Page¶

Start by building a spin-off of the classic "Hello World!" application. When you're finished, the user will be able to get a personal greeting (e.g. "Hello Symfony") by going to the following URL:

在Symfony2中创建一个新的页面是一个简单的过程分为两个步骤：创建一个路由：路由定义的URL（例如，/约）到您的网页，并指定一个控制器（这是一个PHP函数）Symfony2中应该执行时的URL传入的​​请求匹配的路由路径;

创建一个控制器：控制器是一个PHP函数，它接受传入的请求，并把它转换成的返回给用户的Symfony2的Response对象。这种简单的方法就是美丽的，因为它的网络的工作方式相匹配。Web上的每一次互动是通过一个HTTP请求初始化。您的应用程序的任务就是解释请求并返回相应的HTTP响应。

Symfony2中遵循这一理念，为您提供了工具和公约，让您的应用程序组织，因为它生长在用户和复杂性。

环境与前端控制器¶

Symfony的每一个应用程序内的环境中运行。的环境是一组特定的配置和装载的束，用字符串表示。相同的应用程序可以具有不同的结构通过运行该应用程序在不同的环境中运行。Symfony2中自带的定义三种环境 - 开发，测试和督促 - 但你可以创建自己的为好。环境是允许一个应用程序有内置的调试和优化速度生产环境中的开发环境非常有用。您还可以加载基于所选的特定环境捆。例如，Symfony2中自带的WebProfilerBundle（下面描述），使只在开发和测试环境。

Symfony2的带有两个Web访问的前端控制器：app_dev.php提供了开发环境，并app.php提供prod环境。所有的web访问Symfony2的正常穿越这些前端控制器中的一个。（测试环境通常只运行单元测试时使用的，所以没有一个专用的前端控制器的控制台工具还提供了可与任何环境中使用一个前端控制器。）

当前端控制器初始化内核，它提供两个参数：环境，以及在调试模式下的内核是否应该运行。为了使您的应用程序响应速度更快，Symfony2的维护应用程序/缓存/目录下的缓存。当启用了调试模式（如app_dev.php确实在默认情况下），这个缓存是每当你修改任何代码或配置自动刷新。在调试模式下运行时，Symfony2的运行速度较慢，但​​您所做的更改，而不必手动清除缓存反映。

在“你好Symfony的！”页面¶

首先建立一个分拆的经典的“Hello World！”应用程序。当你完成后，用户将可以通过访问以下网址获得个人问候语（如“你好Symfony的”）：

{% highlight PHP %}
http://localhost/app_dev.php/hello/Symfony
{% endhighlight %}

Actually, you'll be able to replace Symfony with any other name to be greeted. To create the page, follow the simple two-step process.

Tips:

The tutorial assumes that you've already downloaded Symfony2 and configured your webserver. The above URL assumes that localhost points to the web directory of your new Symfony2 project. For detailed information on this process, see the documentation on the web server you are using. Here's the relevant documentation page for some web server you might be using:


For Apache HTTP Server, refer to Apache's DirectoryIndex documentation

For Nginx, refer to Nginx HttpCoreModule location documentation

Before you begin: Create the Bundle¶

Before you begin, you'll need to create a bundle. In Symfony2, a bundle is like a plugin, except that all of the code in your application will live inside a bundle.

A bundle is nothing more than a directory that houses everything related to a specific feature, including PHP classes, configuration, and even stylesheets and JavaScript files (see The Bundle System).

To create a bundle called AcmeHelloBundle (a play bundle that you'll build in this chapter), run the following command and follow the on-screen instructions (use all of the default options):

其实，你就可以与被招呼任何其他名称代替Symfony的。要创建该页面，按照简单的两步过程。

温馨提示：

本教程假设您已经下载Symfony2中并配置您的网络服务器。上面的URL假设本地主机指向你的新Symfony2的项目的web目录。有关此过程的详细信息，请参阅您所使用的Web服务器上的文档。下面是你可能会使用一些Web服务器的相关文档页面：

对于Apache HTTP服务器，请参考Apache的文档的DirectoryIndex

nginx的，请参考Nginx的HttpCoreModule位置的文档

开始之前：创建捆绑¶

在开始之前，你需要创建一个包。在Symfony2中，捆绑就像是一个插件，但一切都在你的应用程序的代码会活得包内。

一个bundle无非是，里面的一切关系到一个特定的功能，包括PHP类，配置，甚至是样式表和JavaScript文件（见捆系统）的目录。

要创建一个名为AcmeHelloBundle（一出戏包，你将构建本章中）包，请运行以下命令，然后按照屏幕上的说明（使用所有的默认选项）：

{% highlight PHP %}
$ php app/console generate:bundle --namespace=Acme/HelloBundle --format=yml
{% endhighlight %}

Behind the scenes, a directory is created for the bundle at src/Acme/HelloBundle. A line is also automatically added to the app/AppKernel.php file so that the bundle is registered with the kernel:

{% highlight PHP %}
// app/AppKernel.php
public function registerBundles()
{
    $bundles = array(
        ...,
        new Acme\HelloBundle\AcmeHelloBundle(),
    );
    // ...

    return $bundles;
}
{% endhighlight %}

Now that you have a bundle setup, you can begin building your application inside the bundle.

Step 1: Create the Route¶

By default, the routing configuration file in a Symfony2 application is located at app/config/routing.yml. Like all configuration in Symfony2, you can also choose to use XML or PHP out of the box to configure routes.

If you look at the main routing file, you'll see that Symfony already added an entry when you generated the AcmeHelloBundle:

YAML:

现在，你有一个捆绑的设置，你就可以开始构建你的应用程序的包内。

步骤1：创建路由¶

默认情况下，在Symfony2中应用的路由配置文件位于应用程序/配置/ routing.yml中。像Symfony2的所有配置，你也可以选择使用XML或PHP的开箱即用配置的路由。

如果你看看主路由文件，你会看到，Symfony的已经加入你的时候产生的AcmeHelloBundle的条目：

YAML：

{% highlight PHP %}
# app/config/routing.yml
acme_hello:
    resource: "@AcmeHelloBundle/Resources/config/routing.yml"
    prefix:   /

{% endhighlight %}

This entry is pretty basic: it tells Symfony to load routing configuration from the Resources/config/routing.yml file that lives inside the AcmeHelloBundle. This means that you place routing configuration directly in app/config/routing.yml or organize your routes throughout your application, and import them from here.

Now that the routing.yml file from the bundle is being imported, add the new route that defines the URL of the page that you're about to create:

{% highlight PHP %}
# src/Acme/HelloBundle/Resources/config/routing.yml
hello:
    path:     /hello/{name}
    defaults: { _controller: AcmeHelloBundle:Hello:index }

{% endhighlight %}

The routing consists of two basic pieces: the path, which is the URL that this route will match, and a defaults array, which specifies the controller that should be executed. The placeholder syntax in the path ({name}) is a wildcard. It means that /hello/Ryan, /hello/Fabien or any other similar URL will match this route. The {name} placeholder parameter will also be passed to the controller so that you can use its value to personally greet the user.

Tips:

The routing system has many more great features for creating flexible and powerful URL structures in your application. For more details, see the chapter all about Routing.