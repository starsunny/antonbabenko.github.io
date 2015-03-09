---
layout: post
title:  "Lessons learned from AWS and Varnish"
date:   2015-03-09 21:47:08
categories: tech
---

In this post I want to share a few thoughts I figured out during my latest assignment where I used to design and then implement architecture for [e-commerce web-site for pet owners].

The project was built using Symfony2 PHP framework with rather high demands on code quality as well as performance for end-users.

There was [multi-tier architecture] consisting of several Amazon web-services like:
1) Route 53 (DNS)
2) Cloudfront (CDN)
3) Elastic load-balancer
4) Front-caching layer (EC2 instances with Varnish)
5) Internal elastic load-balancer
6) Application layer (auto-scalling group of stateless web-servers)
7) RDS, Elasticache, etc

The architecture looks rather common and there is a lot of information already online, so I won't go in details here. An interesting and challenging part in the implementation of that architecture was related to the ELB service and how exactly TCP connections handled by AWS services (ELB and Cloudfront).

Error "502 Bad Gateway"
-----------------------

The error I was struggling was "502 Bad Gateway - CloudFront attempted to establish a connection with the origin, but either the attempt failed or the origin closed the connection".

Even after reading such detailed error message I could not find the right combination of values for:
* ELB timeouts and draining connection
* timeout_idle for Varnish
* nginx keep-alive setting

After some time AWS support gave a clear explanation for the problem:

> Varnish apparently sitting with a timeout_idle of 20 seconds, means that Varnish will at times close the TCP connection that the ELB has established to it, while the ELB will continue to think that this connection is still available. If the ELB then tries to send a client request down this connection at a later stage, it will receive an exception, and this will cause the ELB to close the client-side connection with no response.

Sounds clear, I modified timeouts according to their suggestions (most critical piece was to increase timeout_idle in Varnish config from 20 to 60 seconds and it all just works).

BUT! I would not write this post, if it would be just one line fix... :)

Question: Why there is even a connection between ELB and the backend, if I don't make any requests ?
Answer:

> ELB pre-opens TCP connections to the back-ends, even when it does not have a request to dispatch on them. It does this as a performance optimization to reduce latency when a request is eventually available to be dispatched along the TCP connection. The ELB assumed that any TCP connection it has between itself and your back-ends can be left idle for the idle timeout and that the back-end will not close it before this time."

This answer put everything together and it explained why I didn't see an issue while debugging it with ELB excluded.

Also, worth mentioning, Cloudfront on some requests may return cached data completely from the cache, and then ELB is not contacted and therefore backend connection was not established in advance. So, part of our sites were never affected. No ELB = no problem, hehe.

Cloudfront tips
---------------
There are few behavior settings (like headers, cookies and query string forwarding) which can dramatically decrease amount of requests to the backend. If misconfigured then you may serve wrong content to users. Enabled "Query string forwarding" together with Varnish's `sub vcl_hash { hash_data(std.querysort(req.http.x-url)); }` keeps single cached copy in Varnish cache, while bypassing Cloudfront.

Since most of end-users were located in Sweden, where AWS has edge location, by just adding Cloudfront after Route 53 we got average time to connect decreased 80-100ms comparing to time when Route 53 was pointing to ELB. Minor improvement, and for free.

Varnish tips
------------

The biggest performance improvement happened after I revised parts of Varnish configs regarding cookies and ESI, so that average page load time decreased from 1 second per page to 150ms.

Few points on that:
* Try to make an application to serve as much content as possible independently of cookies. Criticise your work!
* Whitelist just accepted cookie names (should be always in-sync with developers and marketing people who like to add something with Google Tag Manager)
* When your application depends on some cookies then consider to put that part of your application info light-weight ESI call, which will do just a little cookie-dependent-magic, while the rest of the page is cookie agnostic (see point 1)
* When your application is setting cookies you ideally should know which urls to allow to set cookies and whitelist them (also require developers awareness)
* If performance is very important, then use [libvmob-cookie] instead of modifying headers with regexps.


Use something what provides an integration with caching proxy, so that developers can control large part of ESI and cookies magic themselves - for PHP there is [FOSHttpCache], which is awesome and it can save a lot of time.


[e-commerce web-site for pet owners]:  http://www.zoozoo.com/
[multi-tier architecture]:             http://en.wikipedia.org/wiki/Multitier_architecture#Three-tier_architecture
[libvmob-cookie]:                      https://github.com/lkarsten/libvmod-cookie
[FOSHttpCache]:                        https://github.com/FriendsOfSymfony/FOSHttpCache