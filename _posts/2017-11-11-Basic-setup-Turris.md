---
layout: post
title:  "Basic setup: Turris Omnia"
date:   2017-11-11 22:55:10 +0100
---

## Introduction

This blog entry documents some unsorted instructions how to setup the popular [Turris Omnia](https://omnia.turris.cz/en/) appliance.

## Prerequsites

* Turris Omnia with OS version 3.7 or higher
* At least completion of the initial setup ([first run](https://www.turris.cz/doc/en/howto/foris))

## Preparation

## zsh

* Install some pre-requisites:
{% highlight bash linenos %}
opkg update && opkg install zsh
{% endhighlight %}
* Change the desired user's shell: vi /etc/passwd
{% highlight bash linenos %}
root:x:0:0:root:/root:/bin/ash to root:x:0:0:root:/root:/bin/zsh
{% endhighlight %}
* Reboot and SSH into the box again

## Sudo user

* SSH into the box (as root)
* Install some pre-requisites:
{% highlight bash linenos %}
opkg update && opkg install sudo shadow-useradd
{% endhighlight %}
* Add a new user, lets say zak (and configure as needed):
{% highlight bash linenos %}
useradd zak
passwd supersecret
mkdir -p /home/zak
chown zak:zak /home/zak
chmod 750 /home/zak
{% endhighlight %}
* Modify the sudoers config as needed: visudo
{% highlight bash linenos %}
# Allow all users in the wheel group to execute any cmd as root
%wheel ALL=(ALL) ALL
{% endhighlight %}
* Add the newly created user to the wheel group:
{% highlight bash linenos %}
# may be the wheel group does not exist
groupadd wheel
usermod -G wheel zak
{% endhighlight %}
* Logout and SSH into box with the new user
* Test if sudo works as expected:
{% highlight bash linenos %}
sudo -l
{% endhighlight %}

## Secure Web Interface
* The following web server config assumes self-signed certificates, which is not recommended for production!
* Install some pre-requisites:
{% highlight bash linenos %}
opkg update && opkg install lighttpd-mod-redirect
{% endhighlight %}
* Modify the following ssl config: sudo vi /etc/lighttpd/conf.d/ssl-enable.conf
{% highlight bash linenos %}
$SERVER["socket"] == ":443" {
        ssl.engine = "enable"
        ssl.pemfile = "/etc/lighttpd-self-signed.pem"
        ssl.use-sslv2 = "disable"
        ssl.use-sslv3 = "disable"
        ssl.cipher-list = "EDH+CAMELLIA:EDH+aRSA:EECDH+aRSA+AESGCM:EECDH+aRSA+SHA256:EECDH:+CAMELLIA128:+AES128:+SSLv3:!aNULL:!eNULL:!LOW:!3DES:!MD5:!EXP:!PSK:!DSS:!RC4:!SEED:!IDEA:!ECDSA:kEDH:CAMELLIA128-SHA:AES128-SHA"
        ssl.honor-cipher-order = "enable"
}

$SERVER["socket"] == "[::]:443" {
        ssl.engine  = "enable"
        ssl.pemfile = "/etc/lighttpd-self-signed.pem"
        ssl.use-sslv2 = "disable"
        ssl.use-sslv3 = "disable"
        ssl.cipher-list = "EDH+CAMELLIA:EDH+aRSA:EECDH+aRSA+AESGCM:EECDH+aRSA+SHA256:EECDH:+CAMELLIA128:+AES128:+SSLv3:!aNULL:!eNULL:!LOW:!3DES:!MD5:!EXP:!PSK:!DSS:!RC4:!SEED:!IDEA:!ECDSA:kEDH:CAMELLIA128-SHA:AES128-SHA"
        ssl.honor-cipher-order = "enable"
}

$HTTP["scheme"] == "https" {
        # Add  'HTTP Strict Transport Security' header (HSTS) to sites
        setenv.add-response-header  += ( "Strict-Transport-Security" => "max-age=31536000; includeSubDomains" )
}
{% endhighlight %}
* Add the following fragment to the web server config (just after the IPv6 listener line): vi /etc/lighttpd/lighttpd.conf
{% highlight bash linenos %}
# Redirect http to https
$HTTP["scheme"] == "http" {
 $HTTP["host"] =~ ".*" {
 url.redirect = (".*" => "https://%0$0")
 }
 setenv.add-environment = (
 "HTTPS" => "on"
 )
}
{% endhighlight %}
* Restart the web server and check if lighttpd is listening on port 80 and 443:
{% highlight bash linenos %}
/etc/init.d/lighttpd restart
netstat -tulpen
{% endhighlight %}
* On a separate client, test the configured ciphers and protocols:
{% highlight bash linenos %}
nmap --script ssl-enum-ciphers -n -p 443 192.168.1.1
{% endhighlight %}
* Open a web browser and enter the router IP or host name without https in order to see if being redirected
