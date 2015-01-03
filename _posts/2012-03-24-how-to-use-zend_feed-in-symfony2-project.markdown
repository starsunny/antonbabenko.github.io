---
layout: post
title:  "Splunk Enterprise on Amazon Linux"
date:   2012-05-19 10:00:15
categories: web-development
---
[Splunk][http://www.splunk.com/] is one of the best tool for log analysis with some flaws in Free edition (no user authentication and 500mb indexing limit per day). Most setups don't need to have user authentication, because splunk can stay behind nginx proxy and have http basic auth there. Also for many sites the limit of 500mb/day is fairy high limit. Though, one of my site has been indexing more than 500mb of logs per day several times per month and according to Splunk Free License it broke the terms of use (huh?), which means that I couldn't even search on already indexed logs.

Splunk Enterprise Trial version works for 60 days and then you have to migrate to Free license (see above) or buy Enterprise license (it is very expensive if you want to have it on small sites like my [imagepush.to][http://imagepush.to]). The solution I found is to reinstall it every 2 months:

{% highlight bash %}
# remove splunk, if you have it installed already
sudo rpm -e splunk

mkdir ~/splunk_tmp
cd ~/splunk_tmp

# Splunk for Amazon Linux (32bit)
curl -O http://216.221.226.44/releases/4.3.2/splunk/linux/splunk-4.3.2-123586.i386.rpm
sudo rpm -U splunk-4.3.2-123586.i386.rpm
sudo /opt/splunk/bin/splunk start
{% endhighlight %}

Then login to splunk web and set password. All settings will be in place already.