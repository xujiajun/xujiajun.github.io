---
layout: post
title: validation [第11天]
date: 2014-05-31 8:31:11
disqus: y
---

Validation is a very common task in web applications. Data entered in forms needs to be validated. Data also needs to be validated before it is written into a database or passed to a web service.

Symfony2 ships with a Validator component that makes this task easy and transparent. This component is based on the JSR303 Bean Validation specification.

The Basics of Validation¶

The best way to understand validation is to see it in action. To start, suppose you've created a plain-old-PHP object that you need to use somewhere in your application:

{% highlight PHP %}
// src/Acme/BlogBundle/Entity/Author.php
namespace Acme\BlogBundle\Entity;

class Author
{
    public $name;
}
{% endhighlight %}
