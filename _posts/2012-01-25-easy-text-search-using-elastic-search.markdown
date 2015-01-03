---
layout: post
title:  "Elastica: reindexing on demand"
date:   2012-03-09 12:02:02
categories: symfony
---

This post was written for Symfony 2.0 long time ago. Things below may not work any more.

I used to make sure that [FOQElasticaBundle] is updating search index after record is created or updated, and there are listeners as specified in [documentation]. It works fine for basic needs out of the box, but I have some index fields, which were populated from another related objects, which I do lazy-fetch, so I ended up with this code:

{% highlight php %}
<?php
// do something with $property
$em->persist($property);
$em->flush();
$em->clear();

// reload property from the database to get saved data with all related objects needed for index
$property = $em->find('MyBundle:Property', $property->getId());

// reindex
$type = $this->kernel->getContainer()->get('foq_elastica.index.website.property');
$modelTransformer = new \FOQ\ElasticaBundle\Transformer\ModelToElasticaAutoTransformer();
$mapping = $type->getMapping();
$fields = array_keys($mapping["property"]["properties"]);

$objectPersister = new \FOQ\ElasticaBundle\Persister\ObjectPersister($type, $modelTransformer, 'My\MyBundle\Entity\Property', $fields);
$objectPersister->replaceOne($property);
{% endhighlight %}

The trick here is to clear doctrine entity manager and then fetch object to replace one more time.

[FOQElasticaBundle]:               https://github.com/Exercise/FOQElasticaBundle
[documentation]: https://github.com/Exercise/FOQElasticaBundle/blob/master/README.md