---
layout: post
title: Performance [第17天]
date: 2014-06-06 8:36:23
disqus: y
---

Symfony2 is fast, right out of the box. Of course, if you really need speed, there are many ways that you can make Symfony even faster. In this chapter, you'll explore many of the most common and powerful ways to make your Symfony application even faster.

Use a Byte Code Cache (e.g. APC)¶
One of the best (and easiest) things that you should do to improve your performance is to use a "byte code cache". The idea of a byte code cache is to remove the need to constantly recompile the PHP source code. There are a number of byte code caches available, some of which are open source. The most widely used byte code cache is probably APC
Using a byte code cache really has no downside, and Symfony2 has been architected to perform really well in this type of environment.
Further Optimizations¶
Byte code caches usually monitor the source files for changes. This ensures that if the source of a file changes, the byte code is recompiled automatically. This is really convenient, but obviously adds overhead.
For this reason, some byte code caches offer an option to disable these checks. Obviously, when disabling these checks, it will be up to the server admin to ensure that the cache is cleared whenever any source files change. Otherwise, the updates you've made won't be seen.
For example, to disable these checks in APC, simply add apc.stat=0 to your php.ini configuration.
Use Composer's Class Map Functionality¶
By default, the Symfony2 standard edition uses Composer's autoloader in the autoload.php file. This autoloader is easy to use, as it will automatically find any new classes that you've placed in the registered directories.
Unfortunately, this comes at a cost, as the loader iterates over all configured namespaces to find a particular file, making file_exists calls until it finally finds the file it's looking for.
The simplest solution is to tell Composer to build a "class map" (i.e. a big array of the locations of all the classes). This can be done from the command line, and might become part of your deploy process:

{% highlight PHP %}
 $ php composer.phar dump-autoload --optimize
{% endhighlight %}

Internally, this builds the big class map array in vendor/composer/autoload_classmap.php.

Caching the Autoloader with APC¶

Another solution is to cache the location of each class after it's located the first time. Symfony comes with a class - ApcClassLoader - that does exactly this. To use it, just adapt your front controller file. If you're using the Standard Distribution, this code should already be available as comments in this file:

{% highlight PHP %}
// app.php
// ...

$loader = require_once __DIR__.'/../app/bootstrap.php.cache';

// Use APC for autoloading to improve performance
// Change 'sf2' by the prefix you want in order
// to prevent key conflict with another application
/*
$loader = new ApcClassLoader('sf2', $loader);
$loader->register(true);
*/

// ...
{% endhighlight %}

For more details, see [Cache a Class Loader](http://symfony.com/doc/current/components/class_loader/cache_class_loader.html).

Tips:


When using the APC autoloader, if you add new classes, they will be found automatically and everything will work the same as before (i.e. no reason to "clear" the cache). However, if you change the location of a particular namespace or prefix, you'll need to flush your APC cache. Otherwise, the autoloader will still be looking at the old location for all classes inside that namespace.