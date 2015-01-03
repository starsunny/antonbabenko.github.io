---
layout: post
title:  "How to use Zend_Feed in Symfony2 project"
date:   2012-03-24 14:40:25
categories: symfony
---

***Update: As of 3.1.2015 there is much nicer approach to generate feeds - [eko/FeedBundle]***

Original article below has been written for Symfony 2.0.

There are [several Symfony2 bundles to generate feeds]. For [imagepush.to] I have to generate a variety of feeds and none of available bundles was good enough, so I want to use Zend_Feed. I didn't want to clone bloated Zend Framework into my project, and luckily KnpLabs have made repos with all popular Zend components already. Here is what I have in **deps**:

{% highlight ini %}
[zend-feed]
    git=http://github.com/KnpLabs/zend-feed.git
    target=Zend/Feed

[zend-loader]
    git=http://github.com/KnpLabs/zend-loader.git
    target=Zend/Loader

[zend-stdlib]
    git=http://github.com/KnpLabs/zend-stdlib.git
    target=Zend/Stdlib
{% endhighlight %}

and this is in <strong>autoload.php</strong>:
{% highlight php %}
<?php
$loader->registerNamespaces(array(
    // ...
    'Zend'            => __DIR__.'/../vendor'
    // ...
));
{% endhighlight %}


[several Symfony2 bundles to generate feeds]:   http://knpbundles.com/search?q=feed
[imagepush.to]:                                 http://imagepush.to/
[eko/FeedBundle]:                               https://github.com/eko/FeedBundle