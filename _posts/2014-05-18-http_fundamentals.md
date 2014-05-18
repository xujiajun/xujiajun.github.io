---
layout: post
title: SYMFONY2和HTTP基本原理 [第1天]
date: 2014-05-18 17:39:31
disqus: y
---

用Symfony2有一段时间了，但没有系统的学习，今天开始我将开始系统全面学习这个full stack PHP framework.

来吧，跟我一起来学习这个强大的框架吧。

恭喜！通过了解Symfony2，对你的朝着成为一个更高效，全面的和流行的Web开发人员（实际上，你对你自己的最后一部分）的方式。Symfony2的是建立在回到基础：开发，让你发展得更快，建设更强大的应用程序的工具，同时保持你的出路。Symfony是建立在许多技术最好的想法：工具和概念，你要学会代表数千人的努力，多年来。换句话说，你不只是学习“Symfony”，你正在学习网页，开发的最佳实践，以及如何使用许多令人惊喜的新的PHP库，Symfony2的内部或独立的基本原则。所以，做好准备。真到了Symfony2的理念，在这一章首先解释常见的Web开发的基本概念：HTTP。不管你的背景或首选的编程语言，这一章是每一个web开发者必读的。

HTTP is simple

HTTP（超文本传输协议的爱好者）是一种文本语言，允许两台机器相互通信。就是这样！例如，检查最新的时XKCD漫画，下面（大约）对话发生：

<img src="/images/symfony2/http-xkcd-step1.png"></img>

Step1: The Client Sends a Request¶

在网络上每次谈话开始于一个请求。该请求是由客户端（例如浏览器，一个iPhone应用程序，等等）在被称为HTTP的一个特殊的格式创建一个文本消息。客户端发送一个请求到服务器，然后等待响应。

In HTTP-speak, this HTTP request would actually look something like this:

{% highlight PHP %}
GET / HTTP/1.1
Host: xkcd.com
Accept: text/html
User-Agent: Mozilla/5.0 (Macintosh)
{% endhighlight %}

这个简单的消息通信一切必要对哪些资源的客户端请求。一个HTTP请求的第一行是最重要的，并包含两个事情：URI和HTTP方法。

Step 2: The Server Returns a Response¶

一旦服务器收到请求时，它知道哪些资源的客户端需要（通过URI）和客户端需要做的是资源（通过方法）是什么。例如，在GET请求的情况下，服务器准备的资源，并返回它在一个HTTP响应。考虑从XKCD Web服务器的响应：

<img src="/images/symfony2/http-xkcd-step2.png"></img>
翻译成HTTP，发回给浏览器的响应将是这个样子：

{% highlight PHP %}
HTTP/1.1 200 OK
Date: Sat, 02 Apr 2011 21:05:05 GMT
Server: lighttpd/1.4.19
Content-Type: text/html
<html>
  <!-- ... HTML for the xkcd comic -->
</html>
{% endhighlight %}

HTTP响应包含所请求的资源（在这种情况下，HTML内容），以及有关的反应的其它信息。第一行是特别重要的，并且包含在HTTP响应状态码（例子中为200）。状态代码进行通信的请求返回给客户端的总体结果。是请求成功？有没有错误？不同的状态代码存在，表明成功，错误，或者客户端需要做一些事情（例如重定向到另一页）。想要知道更多http状态码，这边附上维基百科上的[HTTP状态码](http://en.wikipedia.org/wiki/List_of_HTTP_status_codes)列表的文章。

这边还提到了　HTTP response header is Content-Type　和　multiple different formats　多种格式（like HTML, XML, or JSON ）
the Content-Type header uses Internet Media Types like text/html to tell the client which format is being returned. A list of common media types can be found on Wikipedia's [List of common media types](http://en.wikipedia.org/wiki/Internet_media_type#List_of_common_media_types) article.

Many other headers exist, some of which are very powerful. For example, certain headers can be used to create a powerful caching system.（header还可以用作缓存）

Requests, Responses and Web Development¶

这个请求 - 响应会话是网络上的所有通信的基本过程。
最重要的事实是：不管你使用的语言，应用程序的类型你建立（网络，手机，JSON API），或者你按照发展理念，应用程序的最终目标是始终要了解每个请求，并创建并返回相应的响应。

Symfony的架构也是这个基础上实现的。

tips:

To learn more about the HTTP specification：

read the original [HTTP 1.1 RFC](http://www.w3.org/Protocols/rfc2616/rfc2616.html) or [the HTTP Bis](http://datatracker.ietf.org/wg/httpbis/)

A great tool to check both the request and response headers while browsing is the [Live HTTP Headers](https://addons.mozilla.org/en-US/firefox/addon/live-http-headers/) extension for Firefox.

Requests and Responses in PHP

那么，在PHP中,request和response如何互动?PHP的实现显得有点抽象：

{% highlight PHP %}
$uri = $_SERVER['REQUEST_URI'];
$foo = $_GET['foo'];

header('Content-type: text/html');
echo 'The URI requested is: '.$uri;
echo 'The value of the "foo" parameter is: '.$foo;
{% endhighlight %}

PHP will create a true HTTP response and return it to the client:

{% highlight PHP %}
HTTP/1.1 200 OK
Date: Sat, 03 Apr 2011 02:14:33 GMT
Server: Apache/2.2.17 (Unix)
Content-Type: text/html

The URI requested is: /testing?foo=symfony
The value of the "foo" parameter is: symfony
{% endhighlight%}

Requests and Responses in Symfony

{% highlight PHP %}

use Symfony\Component\HttpFoundation\Request;

$request = Request::createFromGlobals();

// the URI being requested (e.g. /about) minus any query parameters
$request->getPathInfo();

// retrieve GET and POST variables respectively
$request->query->get('foo');
$request->request->get('bar', 'default value if bar does not exist');

// retrieve SERVER variables
$request->server->get('HTTP_HOST');

// retrieves an instance of UploadedFile identified by foo
$request->files->get('foo');

// retrieve a COOKIE value
$request->cookies->get('PHPSESSID');

// retrieve an HTTP request header, with normalized, lowercase keys
$request->headers->get('host');
$request->headers->get('content_type');

$request->getMethod(); // GET, POST, PUT, DELETE, HEAD
$request->getLanguages(); // an array of languages the client accepts

{% endhighlight %}

