---
layout: post
title: Symfony2 VS Flat PHP [第2天]
date: 2014-05-19 7:39:31
disqus: y
---

Why is Symfony2 better than just opening up a file and writing flat PHP?

If you've never used a PHP framework, aren't familiar with the MVC philosophy, or just wonder what all the hype is around Symfony2, this chapter is for you. Instead of telling you that Symfony2 allows you to develop faster and better software than with flat PHP, you'll see for yourself.

In this chapter, you'll write a simple application in flat PHP, and then refactor it to be more organized. You'll travel through time, seeing the decisions behind why web development has evolved over the past several years to where it is now.
By the end, you'll see how Symfony2 can rescue you from mundane tasks and let you take back control of your code.
A Simple Blog in Flat PHP¶
In this chapter, you'll build the token blog application using only flat PHP. To begin, create a single page that displays blog entries that have been persisted to the database. Writing in flat PHP is quick and dirty:

{% highlight PHP %}
<?php
// index.php
$link = mysql_connect('localhost', 'myuser', 'mypassword');
mysql_select_db('blog_db', $link);

$result = mysql_query('SELECT id, title FROM post', $link);
?>

<!DOCTYPE html>
<html>
    <head>
        <title>List of Posts</title>
    </head>
    <body>
        <h1>List of Posts</h1>
        <ul>
            <?php while ($row = mysql_fetch_assoc($result)): ?>
            <li>
                <a href="/show.php?id=<?php echo $row['id'] ?>">
                    <?php echo $row['title'] ?>
                </a>
            </li>
            <?php endwhile; ?>
        </ul>
    </body>
</html>

<?php
mysql_close($link);
?>
{% endhighlight %}

That's quick to write, fast to execute, and, as your app grows, impossible to maintain. There are several problems that need to be addressed:

No error-checking: What if the connection to the database fails?
Poor organization: If the application grows, this single file will become increasingly unmaintainable. Where should you put code to handle a form submission? How can you validate data? Where should code go for sending emails?
Difficult to reuse code: Since everything is in one file, there's no way to reuse any part of the application for other "pages" of the blog.

Tips:

Another problem not mentioned here is the fact that the database is tied to MySQL. Though not covered here, Symfony2 fully integrates Doctrine, a library dedicated to database abstraction and mapping.

Isolating the Presentation¶

The code can immediately gain from separating the application "logic" from the code that prepares the HTML "presentation":

{% highlight PHP %}
<?php
// index.php
$link = mysql_connect('localhost', 'myuser', 'mypassword');
mysql_select_db('blog_db', $link);

$result = mysql_query('SELECT id, title FROM post', $link);

$posts = array();
while ($row = mysql_fetch_assoc($result)) {
    $posts[] = $row;
}

mysql_close($link);

// include the HTML presentation code
require 'templates/list.php';
?>
{% endhighlight %}

The HTML code is now stored in a separate file (templates/list.php), which is primarily an HTML file that uses a template-like PHP syntax:

{% highlight PHP %}
<!DOCTYPE html>
<html>
    <head>
        <title>List of Posts</title>
    </head>
    <body>
        <h1>List of Posts</h1>
        <ul>
            <?php foreach ($posts as $post): ?>
            <li>
                <a href="/read?id=<?php echo $post['id'] ?>">
                    <?php echo $post['title'] ?>
                </a>
            </li>
            <?php endforeach; ?>
        </ul>
    </body>
</html>
{% endhighlight %}

By convention, the file that contains all of the application logic - index.php - is known as a "controller". The term controller is a word you'll hear a lot, regardless of the language or framework you use. It refers simply to the area of your code that processes user input and prepares the response.

In this case, the controller prepares data from the database and then includes a template to present that data. With the controller isolated, you could easily change just the template file if you needed to render the blog entries in some other format (e.g. list.json.php for JSON format).

Isolating the Application (Domain) Logic¶

So far the application contains only one page. But what if a second page needed to use the same database connection, or even the same array of blog posts? Refactor the code so that the core behavior and data-access functions of the application are isolated in a new file called model.php:

{% highlight PHP %}
<?php
// model.php
function open_database_connection()
{
    $link = mysql_connect('localhost', 'myuser', 'mypassword');
    mysql_select_db('blog_db', $link);

    return $link;
}

function close_database_connection($link)
{
    mysql_close($link);
}

function get_all_posts()
{
    $link = open_database_connection();

    $result = mysql_query('SELECT id, title FROM post', $link);
    $posts = array();
    while ($row = mysql_fetch_assoc($result)) {
        $posts[] = $row;
    }
    close_database_connection($link);

    return $posts;
}
?>
{% endhighlight %}

Tips:

The filename model.php is used because the logic and data access of an application is traditionally known as the "model" layer. In a well-organized application, the majority of the code representing your "business logic" should live in the model (as opposed to living in a controller). And unlike in this example, only a portion (or none) of the model is actually concerned with accessing a database.

The controller (index.php) is now very simple:

{% highlight PHP %}
<?php
require_once 'model.php';

$posts = get_all_posts();

require 'templates/list.php';
?>
{% endhighlight %}

Now, the sole task of the controller is to get data from the model layer of the application (the model) and to call a template to render that data. This is a very simple example of the model-view-controller pattern.

Isolating the Layout¶

At this point, the application has been refactored into three distinct pieces offering various advantages and the opportunity to reuse almost everything on different pages.
The only part of the code that can't be reused is the page layout. Fix that by creating a new layout.php file:

{% highlight PHP %}
<!-- templates/layout.php -->
<!DOCTYPE html>
<html>
    <head>
        <title><?php echo $title ?></title>
    </head>
    <body>
        <?php echo $content ?>
    </body>
</html>
{% endhighlight %}

The template (templates/list.php) can now be simplified to "extend" the layout:

{% highlight PHP %}
<?php $title = 'List of Posts' ?>

<?php ob_start() ?>
    <h1>List of Posts</h1>
    <ul>
        <?php foreach ($posts as $post): ?>
        <li>
            <a href="/read?id=<?php echo $post['id'] ?>">
                <?php echo $post['title'] ?>
            </a>
        </li>
        <?php endforeach; ?>
    </ul>
<?php $content = ob_get_clean() ?>

<?php include 'layout.php' ?>
{% endhighlight %}

You've now introduced a methodology that allows for the reuse of the layout. Unfortunately, to accomplish this, you're forced to use a few ugly PHP functions (ob_start(), ob_get_clean()) in the template. Symfony2 uses a Templating component that allows this to be accomplished cleanly and easily. You'll see it in action shortly.

Adding a Blog "show" Page¶


The blog "list" page has now been refactored so that the code is better-organized and reusable. To prove it, add a blog "show" page, which displays an individual blog post identified by an id query parameter.
To begin, create a new function in the model.php file that retrieves an individual blog result based on a given id:


{% highlight PHP %}
<?php
// model.php
function get_post_by_id($id)
{
    $link = open_database_connection();

    $id = intval($id);
    $query = 'SELECT date, title, body FROM post WHERE id = '.$id;
    $result = mysql_query($query);
    $row = mysql_fetch_assoc($result);

    close_database_connection($link);

    return $row;
}
?>
{% endhighlight %}

Next, create a new file called show.php - the controller for this new page:

{% highlight PHP %}
<?php
require_once 'model.php';

$post = get_post_by_id($_GET['id']);

require 'templates/show.php';
?>
{% endhighlight %}

Finally, create the new template file - templates/show.php - to render the individual blog post:

{% highlight PHP %}
<?php $title = $post['title'] ?>

<?php ob_start() ?>
    <h1><?php echo $post['title'] ?></h1>

    <div class="date"><?php echo $post['date'] ?></div>
    <div class="body">
        <?php echo $post['body'] ?>
    </div>
<?php $content = ob_get_clean() ?>

<?php include 'layout.php' ?>
{% endhighlight %}

Creating the second page is now very easy and no code is duplicated. Still, this page introduces even more lingering problems that a framework can solve for you. For example, a missing or invalid id query parameter will cause the page to crash. It would be better if this caused a 404 page to be rendered, but this can't really be done easily yet. Worse, had you forgotten to clean the id parameter via the intval() function, your entire database would be at risk for an SQL injection attack.

Another major problem is that each individual controller file must include the model.php file. What if each controller file suddenly needed to include an additional file or perform some other global task (e.g. enforce security)? As it stands now, that code would need to be added to every controller file. If you forget to include something in one file, hopefully it doesn't relate to security...


A "Front Controller" to the Rescue¶

The solution is to use a front controller: a single PHP file through which all requests are processed. With a front controller, the URIs for the application change slightly, but start to become more flexible:

{% highlight PHP %}
Without a front controller
/index.php          => Blog post list page (index.php executed)
/show.php           => Blog post show page (show.php executed)

With index.php as the front controller
/index.php          => Blog post list page (index.php executed)
/index.php/show     => Blog post show page (index.php executed)
{% endhighlight %}






