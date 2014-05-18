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

Like HTTP itself, the Request and Response objects are pretty simple. The hard part of building an application is writing what comes in between.换句话讲，中间的过程才是复杂的。

Stay Organized

对于路由“丑陋”写法：

{% highlight PHP %}
// index.php
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;

$request = Request::createFromGlobals();
$path = $request->getPathInfo(); // the URI path being requested

if (in_array($path, array('', '/'))) {
    $response = new Response('Welcome to the homepage.');
} elseif ($path == '/contact') {
    $response = new Response('Contact us');
} else {
    $response = new Response('Page not found.', Response::HTTP_NOT_FOUND);
}
$response->send();

{% endhighlight %}

the Symfony Application Flow

When you let Symfony handle each request, life is much easier. Symfony follows the same simple pattern for every request:
<img src="/images/symfony2/request-flow.png"></img>
传入的请求被路由解释并传递给控制器，返回函数响应的对象。

您的网站的每个“页”是在不同的URL映射到不同的PHP功能的路由配置文件中定义。每个PHP函数的工作，称为控制器，是利用信息从请求-以及许多其他工具symfony让可用-创建并返回一个响应 对象。换句话说，控制器就是你的代码有云：这就是你理解的请求，并创建一个响应。

就是这么简单！
每个请求执行一个前端控制器文件;
路由系统决定了哪些PHP函数应该基于你所创建的请求和路由的配置信息来执行;
正确的PHP函数被执行时，在您的代码创建并返回相应的响应对象。

A Symfony Request in Action

routing configuration file like :
{% highlight YAML %}
YAML:
# app/config/routing.yml
contact:
    path:     /contact
    defaults: { _controller: AcmeDemoBundle:Main:contact }

{% endhighlight %}

MainController:

{% highlight PHP %}
// src/Acme/DemoBundle/Controller/MainController.php
namespace Acme\DemoBundle\Controller;

use Symfony\Component\HttpFoundation\Response;

class MainController
{
    public function contactAction()
    {
        return new Response('<h1>Contact us!</h1>');
    }
}
{% endhighlight %}

Symfony2的：建立你的应用程序，而不是你的工具。¶

现在你知道的任何应用程序的目标是解释每个传入的请求，并创建相应的响应。作为应用的增长，它变得更加困难，以保持你的代码组织和维护。不变的是，同样复杂的任务滚滚而来了一遍又一遍：坚持的东西到数据库，渲染和重复使用的模板，处理表单提交，发送电子邮件，验证用户输入和处理的安全性。
好消息是，没有这些问题是独一无二的。symfony提供了一个完整的工具，允许你建立你的应用程序，而不是你的工具的框架。

独立的工具：在Symfony2的组件¶：

HttpFoundation -包含请求和响应类，以及其他类处理会话和文件上传;

路由 -强大和快速的路由系统，它允许您将特定的URI（例如映射/接触），以如何的请求应该被处理（例如，执行一些信息contactAction（） 方法）;

形式 -用于创建表单和处理表单提交一个全功能和灵活的框架;

验证器 -一种用于创建有关数据的规则，然后验证用户提交的数据是否遵循这些规则;

类加载器 -一个自动加载库，允许无需手动使用PHP类需要包含这些类文件;

模板 -一个工具包，用于呈现模板，搬运模板继承（即模板装饰布局），并执行其他常见的模板任务;

安全性 -一个强大的库，用于处理所有类型的安全的应用程序中;

翻译 -在您的应用程序翻译字符串的框架。

这些组件的每个人都被分离，并可以用于任何 PHP项目，无论是否使用Symfony2的框架。每一部分是由在需要更换和必要时可以使用。

整的解决方案：Symfony2的框架¶

那么，什么是对Symfony2框架？在Symfony2的框架是一个PHP库，实现了两个不同的任务：
提供了一个选择的组件（即Symfony2的组件）和第三方库（例如的雨燕梅勒用于发送电子邮件）;
提供合理的配置和“胶水”库，结合所有这些作品一起。
该框架的目标是整合多个独立的工具，以便为开发人员提供一致的体验。甚至框架本身是一个Symfony2的束（即插件buddle），可配置或更换完全。
Symfony2中提供了强大的工具集，用于快速开发Web应用程序而不强加在你的应用程序。普通用户可以通过使用Symfony2的分布，它提供了一个项目骨架合理的默认值开始迅速开发。对于高级玩家，the sky is the limit。

translate by Xujiajun

From:[Symfony.com](http://symfony.com/doc/current/book/http_fundamentals.html#the-full-solution-the-symfony2-framework)