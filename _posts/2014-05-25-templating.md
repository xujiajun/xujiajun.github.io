---
layout: post
title: Creating and Using Templates [第7天]
date: 2014-05-25 7:39:31
disqus: y
---

As you know, the controller is responsible for handling each request that comes into a Symfony2 application. In reality, the controller delegates most of the heavy work to other places so that code can be tested and reused. When a controller needs to generate HTML, CSS or any other content, it hands the work off to the templating engine. In this chapter, you'll learn how to write powerful templates that can be used to return content to the user, populate email bodies, and more. You'll learn shortcuts, clever ways to extend templates and how to reuse template code.