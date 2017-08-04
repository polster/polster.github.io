---
layout: post
title:  "Pi-Hole on Turris Omnia"
date:   2017-08-04 22:55:10 +0100
---

## Introduction

This blog entry summarizes the needed steps (may be not all) how to setup the popular Add-ware blocker [Pi-Hole](https://pi-hole.net/) on the [Turris Omnia](https://omnia.turris.cz/en/) appliance.

## Prerequsites

* Turris Omnia with OS version 3.7 or higher
* At least completion of the initial setup ([first run](https://www.turris.cz/doc/en/howto/foris))

## Container Setup

* SSH into the box
* Create a new lxc container:
{% highlight bash linenos %}
lxc-create -t download -n pi-hole
{% endhighlight %}
* When prompted, enter the following parameters: ubuntu xenial armv7
* Configure a static IP address to be set as new DNS server in a further step:
{% highlight bash linenos %}
vi /srv/lxc/pi-hole/config

# Append the following lines:
lxc.network.ipv4.ipv4 = <static ip address>/<mask bits>
lxc.network.ipv4.gateway = <router address>
{% endhighlight %}
* Configure the container to be started automatically (after reboot):
{% highlight bash linenos %}
vi /etc/config/lxc-auto

# Add or append the following lines
config container
        option name pi-hole
        option timeout 60
{% endhighlight %}
* Start the newly created lxc container:
{% highlight bash linenos %}
lxc-start -n pi-hole
{% endhighlight %}
* Verify if the container is up and running:
{% highlight bash linenos %}
lxc-info -n pi-hole
{% endhighlight %}

![Pi-Hole container]({{ site.url }}/images/turris-lxc-pi-hole-container.png)

## Pi-Hole Setup

* Attach to the container:
{% highlight bash linenos %}
lxc-attach -n pi-hole
{% endhighlight %}
* Install needed package:
{% highlight bash linenos %}
apt install -y wget
{% endhighlight %}
* Download the Pi-Hole installation script and run it:
{% highlight bash linenos %}
wget -O basic-install.sh https://install.pi-hole.net
bash basic-install.sh
{% endhighlight %}
* Follow the script's interactive installation instructions (correct settings should be suggested by the installation assistant)
* As a first test, try to open the Pi-Hole URL http://<pi-hole IP or host name>/admin in a new browser window:

![Pi-Hole container]({{ site.url }}/images/turris-lxc-pi-hole-web-interface.png)

## Turris Setup

* Login to LuCI
* Go to Network interfaces > LAN > DHCP Server > DHCP Option and change IP to the pi-hole container's static IP

## Verification

* Ensure your IP address lease has been renewed on the client from where you are connected to the Turris Omnia box (in order to use Pi-Hole as the DNS server)
* Open the Pi-Hole Add-ware testing page in a new browser window: https://pi-hole.net/pages-to-test-ad-blocking-performance/
* Check if the statistics/counters have changed:

![Pi-Hole container]({{ site.url }}/images/turris-lxc-pi-hole-web-interface2.png)
