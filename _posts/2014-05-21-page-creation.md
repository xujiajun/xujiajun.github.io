---
layout: post
title: Creating Pages in Symfony2 [第4天]
date: 2014-05-20 7:39:31
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