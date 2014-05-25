---
layout: post
title: Controller [第5天]
date: 2014-05-22 7:39:31
disqus: y
---

A controller is a PHP function you create that takes information from the HTTP request and constructs and returns an HTTP response (as a Symfony2 Response object). The response could be an HTML page, an XML document, a serialized JSON array, an image, a redirect, a 404 error or anything else you can dream up. The controller contains whatever arbitrary logic your application needs to render the content of a page.
See how simple this is by looking at a Symfony2 controller in action. The following controller would render a page that simply prints Hello world!:

{% highlight PHP %}

use Symfony\Component\HttpFoundation\Response;

public function helloAction()
{
    return new Response('Hello world!');
}

{% endhighlight %}

The goal of a controller is always the same: create and return a Response object. Along the way, it might read information from the request, load a database resource, send an email, or set information on the user's session. But in all cases, the controller will eventually return the Response object that will be delivered back to the client.

There's no magic and no other requirements to worry about! Here are a few common examples:

Controller A prepares a Response object representing the content for the homepage of the site.

Controller B reads the slug parameter from the request to load a blog entry from the database and create a Response object displaying that blog. If the slug can't be found in the database, it creates and returns a Response object with a 404 status code.

Controller C handles the form submission of a contact form. It reads the form information from the request, saves the contact information to the database and emails the contact information to the webmaster. Finally, it creates a Response object that redirects the client's browser to the contact form "thank you" page.

Requests, Controller, Response Lifecycle¶

Every request handled by a Symfony2 project goes through the same simple lifecycle. The framework takes care of the repetitive tasks and ultimately executes a controller, which houses your custom application code:

Each request is handled by a single front controller file (e.g. app.php or app_dev.php) that bootstraps the application;

The Router reads information from the request (e.g. the URI), finds a route that matches that information, and reads the _controller parameter from the route;
The controller from the matched route is executed and the code inside the controller creates and returns a Response object;

The HTTP headers and content of the Response object are sent back to the client.
Creating a page is as easy as creating a controller (#3) and making a route that maps a URL to that controller (#2).

Tips:

Though similarly named, a "front controller" is different from the "controllers" talked about in this chapter. A front controller is a short PHP file that lives in your web directory and through which all requests are directed. A typical application will have a production front controller (e.g. app.php) and a development front controller (e.g. app_dev.php). You'll likely never need to edit, view or worry about the front controllers in your application.

A Simple Controller¶

While a controller can be any PHP callable (a function, method on an object, or a Closure), in Symfony2, a controller is usually a single method inside a controller object. Controllers are also called actions.

{% highlight PHP %}

// src/Acme/HelloBundle/Controller/HelloController.php
namespace Acme\HelloBundle\Controller;

use Symfony\Component\HttpFoundation\Response;

class HelloController
{
    public function indexAction($name)
    {
        return new Response('<html><body>Hello '.$name.'!</body></html>');
    }
}

{% endhighlight %}


Tips:

Note that the controller is the indexAction method, which lives inside a controller class (HelloController). Don't be confused by the naming: a controller class is simply a convenient way to group several controllers/actions together. Typically, the controller class will house several controllers/actions (e.g. updateAction, deleteAction, etc).

This controller is pretty straightforward:

line 4: Symfony2 takes advantage of PHP 5.3 namespace functionality to namespace the entire controller class. The use keyword imports the Response class, which the controller must return.

line 6: The class name is the concatenation of a name for the controller class (i.e. Hello) and the word Controller. This is a convention that provides consistency to controllers and allows them to be referenced only by the first part of the name (i.e. Hello) in the routing configuration.

line 8: Each action in a controller class is suffixed with Action and is referenced in the routing configuration by the action's name (index). In the next section, you'll create a route that maps a URI to this action. You'll learn how the route's placeholders ({name}) become arguments to the action method ($name).
line 10: The controller creates and returns a Response object.

