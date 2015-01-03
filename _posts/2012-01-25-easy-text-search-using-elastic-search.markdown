---
layout: post
title:  "Easy text search using Elastic Search"
date:   2012-01-25 11:12:08
categories: symfony
---

This post was written for Symfony 2.0 long time ago. Things below may not work any more.

Recently I used to add very plain text search to [ne.no] and since I'm using [Elastic Search] with [FOQElasticaBundle] it was easy to achieve.

{% highlight php %}
<?php
$textQuery = new \Elastica_Query_QueryString();
$textQuery->setFields(array("address", "name", "city", "searchPartNames", "searchCompanyName"));
$textQuery->setDefaultOperator("AND");
$textQuery->setQueryString($selectedText);

$tmpTexts = array();
foreach (explode(" ", $selectedText) as $oneWord) {
    $oneWord = trim($oneWord);
    if ($oneWord == "")
        continue;
    if (preg_match("/^[0-9]+/", $oneWord)) {
        $tmpTexts[] = $oneWord;
    } else {
        $tmpTexts[] = $oneWord . "*";
    }
}
$selectedText = implode(" ", $tmpTexts);
{% endhighlight %}

Added star-wildcard after all words, but not after numbers, so that user is able to find correct building address located on Drammensgt or Drammensvn if he searches for "Drammens 171".

[ne.no]:              http://ne.no/
[Elastic Search]:     http://www.elasticsearch.com/
[FOQElasticaBundle]:  https://github.com/Exercise/FOQElasticaBundle
