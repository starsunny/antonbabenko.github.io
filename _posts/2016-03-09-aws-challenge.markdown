---
layout: post
title:  "AWS Challenge #1"
date:   2016-03-09 12:25:08
categories: tech, aws
---

Yesterday we had our regular [AWS User Group Norway meetup](https://meetup.com/AWS-User-Group-Norway/) where I wanted to bring audience closer to real-life problems solution architects and developers work with daily.

Let me assure that not all challenges I come up with will have practical need or follow so-called "best-practices", but they will certainly give some brain-food for participants who are curios about AWS in one or another way.

For the beginning I challenged audience with this task:

> There is LAMP web-site running on spot instances (within auto-scaling group) where users register accounts and expect confirmation code to be emailed to them as soon as possible.

> There are several other tasks to be performed when user registers an account, so using [Asynchronous Queuing SOA pattern](http://soapatterns.org/design_patterns/asynchronous_queuing) is a good idea.

> We also can't send emails in same request when user register account.

> **Question:** How using SQS and some other AWS services we can achieve the following goals:

> 1. user receives confirmation letter as soon as possible;
> 2. no messages should be lost anytime;
> 3. solution should be highly available, cheap and horizontally scalable;
> 4. there can be [0-inf) visitors during a day.

We will discuss available solutions and ideas during [our next AWS meetup - April 25, 2016](http://www.meetup.com/AWS-User-Group-Norway/events/229257676/).

Feel free to leave a comment to this blog post or ask for clarification.