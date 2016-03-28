---
layout: "post"
title: "Apache - Simple monitoring with Server Status"
date: "2016-02-19 23:26:17 +0100"
---

## Enable mod_status

* Open the file /etc/httpd/conf/httpd.conf
* Make sure the following line looks as follows:
{% highlight text %}
  LoadModule status_module modules/mod_status.so
{% endhighlight %}

## Configure mod_status

* Append the following fragment (adjust as needed) to the vhost file that should be monitored:
{% highlight text %}
  <Location /server-status>
    SetHandler server-status
    Order allow,deny
    Deny from all
    Allow from all
  </Location>
{% endhighlight %}

## Enable extended status

* Enabling the extended status will add more information to the statistics page like CPU usage, requests per second, total traffic and so on
* Edit the httpd.conf file and make sure the following line looks as follows:
{% highlight text %}
  ExtendedStatus On
{% endhighlight %}

## Access the status page

* The status page can be accessed like so:
{% highlight text %}
  http://myhost/server-status
{% endhighlight %}

## Status page via shell

* Instead of accessing the status page via web browser the command line browser lynx may be used alternatively, for instance to track the status via script and store it in a file
* For this make sure lynx is installed
* Sample call:
{% highlight text %}
  lynx http://myhost/server-status
{% endhighlight %}
