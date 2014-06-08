---
layout: post
title: Internals [第18天]
date: 2014-06-08 8:35:37
disqus: y
---

Looks like you want to understand how Symfony2 works and how to extend it. That makes me very happy! This section is an in-depth explanation of the Symfony2 internals.

Tips:

You need to read this section only if you want to understand how Symfony2 works behind the scene, or if you want to extend Symfony2.

Overview

The Symfony2 code is made of several independent layers. Each layer is built on top of the previous one.

Tips:

Autoloading is not managed by the framework directly; it's done by using Composer's autoloader (vendor/autoload.php), which is included in the app/autoload.php file.

HttpFoundation Component

The deepest level is the HttpFoundation component. HttpFoundation provides the main objects needed to deal with HTTP. It is an object-oriented abstraction of some native PHP functions and variables:

The Request class abstracts the main PHP global variables like $_GET, $_POST, $_COOKIE, $_FILES, and $_SERVER;

The Response class abstracts some PHP functions like header(), setcookie(), and echo;

The Session class and SessionStorageInterface interface abstract session management session_*() functions.

HttpKernel Component¶

On top of HttpFoundation is the HttpKernel component. HttpKernel handles the dynamic part of HTTP; it is a thin wrapper on top of the Request and Response classes to standardize the way requests are handled. It also provides extension points and tools that makes it the ideal starting point to create a Web framework without too much overhead.

It also optionally adds configurability and extensibility, thanks to the DependencyInjection component and a powerful plugin system (bundles).

Read more about the [HttpKernel component](http://symfony.com/doc/current/components/http_kernel/introduction.html), Dependency Injection and [Bundles](http://symfony.com/doc/current/cookbook/bundles/best_practices.html).

FrameworkBundle¶

The FrameworkBundle bundle is the bundle that ties the main components and libraries together to make a lightweight and fast MVC framework. It comes with a sensible default configuration and conventions to ease the learning curve.

Kernel¶

The HttpKernel class is the central class of Symfony2 and is responsible for handling client requests. Its main goal is to "convert" a Request object to a Response object.


