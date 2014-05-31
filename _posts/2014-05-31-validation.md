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

So far, this is just an ordinary class that serves some purpose inside your application. The goal of validation is to tell you whether or not the data of an object is valid. For this to work, you'll configure a list of rules (called constraints) that the object must follow in order to be valid. These rules can be specified via a number of different formats (YAML, XML, annotations, or PHP).

For example, to guarantee that the $name property is not empty, add the following:

Yaml:

{% highlight PHP %}
# src/Acme/BlogBundle/Resources/config/validation.yml
Acme\BlogBundle\Entity\Author:
    properties:
        name:
            - NotBlank: ~
{% endhighlight %}

Tips:

Protected and private properties can also be validated, as well as "getter" methods (see Constraint Targets).

Using the validator Service¶

Next, to actually validate an Author object, use the validate method on the validator service (class Validator). The job of the validator is easy: to read the constraints (i.e. rules) of a class and verify whether or not the data on the object satisfies those constraints. If validation fails, a non-empty list of errors (class ConstraintViolationList) is returned. Take this simple example from inside a controller:

{% highlight PHP %}
// ...
use Symfony\Component\HttpFoundation\Response;
use Acme\BlogBundle\Entity\Author;

public function indexAction()
{
    $author = new Author();
    // ... do something to the $author object

    $validator = $this->get('validator');
    $errors = $validator->validate($author);

    if (count($errors) > 0) {
        /*
         * Uses a __toString method on the $errors variable which is a
         * ConstraintViolationList object. This gives us a nice string
         * for debugging
         */
        $errorsString = (string) $errors;

        return new Response($errorsString);
    }

    return new Response('The author is valid! Yes!');
}
{% endhighlight %}

If the $name property is empty, you will see the following error message:

{% highlight PHP %}
Acme\BlogBundle\Author.name:
    This value should not be blank
{% endhighlight %}

If you insert a value into the name property, the happy success message will appear.

Tips:

Most of the time, you won't interact directly with the validator service or need to worry about printing out the errors. Most of the time, you'll use validation indirectly when handling submitted form data. For more information, see the Validation and Forms.

You could also pass the collection of errors into a template.

{% highlight PHP %}
if (count($errors) > 0) {
    return $this->render('AcmeBlogBundle:Author:validate.html.twig', array(
        'errors' => $errors,
    ));
}
{% endhighlight %}









