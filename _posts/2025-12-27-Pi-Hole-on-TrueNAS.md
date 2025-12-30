---
layout: post
title: "Pi-Hole on TrueNAS"
date: 2025-12-27 05:55:10 +0100
last_modified_at: 2025-12-30T00:21:37
---

## Introduction

This blog entry outlines the steps to setup Pi-Hole together with Bind9 on TrueNAS Scale, making use of its default container runtime (LXC).

## Prerequisites

* Up and running TrueNAS Scale

## Pi-Hole Setup

Login to TrueNAS web console, go to Datasets and create a new dataset named `pi-hole` for the Pi-Hole app - assuming one dataset per app. Depending on the setup, you may need to SSH into the TrueNAS host and set permissions accordingly.

A file and folder structure may look like this:
{% highlight bash linenos %}
/mnt/apps/pi-hole
/mnt/apps/pi-hole/config
/mnt/apps/pi-hole/compose.yaml
{% endhighlight %}

The content of the `compose.yaml` file may look like this:
{% highlight yaml linenos %}
services:

  pi-hole:
    image: pihole/pihole:2025.11.1
    ports:
      # DNS
      - "192.168.1.170:53:53/tcp"
      - "192.168.1.170:53:53/udp"
    networks:
      - nginx
      - dns
    environment:
      PIHOLE_UID: ${PUID:-950}
      PIHOLE_GID: ${PGID:-950}
      TZ: 'Europe/Zurich'
      # If using Docker's default `bridge` network setting the dns listening mode should be set to 'all'
      FTLCONF_dns_listeningMode: all
      # Sets upstream DNS to internal bind9 dns server
      FTLCONF_dns_upstreams: bind9
    volumes:
      - './config/pi-hole:/etc/pihole'
    restart: always
    cap_add:
      # See https://github.com/pi-hole/docker-pi-hole#note-on-capabilities
      # Optional, if Pi-hole should get some more processing time
      - SYS_NICE

networks:
  nginx:
    external: true
  dns:
    external: true
{% endhighlight %}

Explanations to be well noted:
* The user/group to start the container process and have at least read permission to the dataset, can be set through the PUID and PGID env var, or use default values.
* Access to the Pi-Hole web interface is provided through the `nginx` network, respectively via Nginx reverse proxy.
* DNS forwarding to an internal Bind9 DNS server goes through the `dns` network.
* Using the `FTLCONF_dns_upstreams` env var, the internal Bind9 DNS server is configured as upstream DNS server for Pi-Hole.
* The networks are assumed external, to be created by the concerned apps.
* Because of more than one IP address configured with the network interface, the ports binding also includes an explicit IP address to bind to.

To deploy and run Pi-Hole like a TrueNAS app, go to the `Apps` section in the TrueNAS web console, click `Discover Apps`, then click on the three dots on the upper right and click `Install via YAML`.

Insert the name of the app (e.g. pi-hole) and copy/paste the following custom configuration (adjust path if needed):
{% highlight yaml linenos %}
include:
  - /mnt/apps/pi-hole/compose.yaml
{% endhighlight %}

TrueNAS should do the deployment and indicate once the app is running.

## Bind9 Setup

The steps to deploy and run Bind9 as an internal DNS server, are very similar to the ones for Pi-Hole.

After creating a new dataset named `bind9` and adjusting permissions, a few config files need to to be created:
{% highlight bash linenos %}
/mnt/apps/bind9
/mnt/apps/bind9/config
/mnt/apps/bind9/config/my.domain.io.zone
/mnt/apps/bind9/config/named.conf
/mnt/apps/bind9/config/named.conf.options
/mnt/apps/bind9/config/named.conf.local
/mnt/apps/bind9/config/named.conf.logging
/mnt/apps/bind9/compose.yaml
{% endhighlight %}

named.conf:
{% highlight bash linenos %}
include "/etc/bind/named.conf.options";
include "/etc/bind/named.conf.local";
include "/etc/bind/named.conf.logging";
{% endhighlight %}

named.conf.options:
{% highlight bash linenos %}
// only internal addresses
acl internal {
    192.168.1.0/24; // internal subnet
    172.16.0.0/16; // Docker subnet
	localhost;
	localnets;
};

options {

    directory "/var/cache/bind";

    forwarders {
        1.1.1.1;
        1.0.0.1;
    };

    allow-query { internal; };
    allow-recursion { internal; };
    allow-query-cache { internal; };

    dnssec-validation no;
};
{% endhighlight %}

named.conf.local:
{% highlight bash linenos %}
zone "my.domain.io" IN {
    type master;
    file "/etc/bind/my.domain.io.zone";
};
{% endhighlight %}

named.conf.logging:
{% highlight bash linenos %}
logging {
  channel stdout_channel {
    stderr;
    severity info;
    print-category yes;
    print-severity yes;
    print-time yes;
  };
  category default { stdout_channel; };
  category queries { stdout_channel; };
  category security { stdout_channel; };
  category dnssec { stdout_channel; };
};
{% endhighlight %}

compose.yaml:
{% highlight yaml linenos %}
services:

  bind9:
    image: ubuntu/bind9:latest
    environment:
      - TZ=Europe/Zurich
    ports:
      - "192.168.1.170:5453:53/tcp"
      - "192.168.1.170:5453:53/udp"
    volumes:
      - ./config/bind9/named.conf:/etc/bind/named.conf
      - ./config/bind9/my.domain.io.zone:/etc/bind/my.domain.io.zone
      - ./config/bind9/named.conf.options:/etc/bind/named.conf.options
      - ./config/bind9/named.conf.logging:/etc/bind/named.conf.logging
      - ./config/bind9/named.conf.local:/etc/bind/named.conf.local
      - bind9_cache:/var/cache/bind
      - bind9_records:/var/lib/bind
    restart: always

volumes:
  bind9_cache:
  bind9_records:

networks:
  default:
    name: dns
{% endhighlight %}

Insert the name of the app (e.g. pi-hole) and copy/paste the following custom configuration (adjust path if needed):
{% highlight yaml linenos %}
include:
  - /mnt/apps/pi-hole/compose.yaml
{% endhighlight %}

Like before, go to the `Apps` section in the TrueNAS web console, click `Discover Apps`, then click on the three dots on the upper right and click `Install via YAML`.

Insert the name of the app (e.g. bind9) and copy/paste the following custom configuration (adjust path if needed):
{% highlight yaml linenos %}
include:
  - /mnt/apps/bind9/compose.yaml
{% endhighlight %}

TrueNAS should do the deployment and indicate once the app is running.

## Verification

To verify that the setup is working, use `dig` command to query at different levels.
{% highlight bash linenos %}
# Query Bind9 directly
dig @192.168.1.170 -p 5353 my.domain.io

# Query Pi-Hole
dig @192.168.1.170 -p 53 my.domain.io
{% endhighlight %}

Once verified, configure the local DHCP server to use the Pi-Hole IP address as DNS server. Then, renew the IP address lease and test from any client device if DNS resolution works.

## Conclusion

With this setup, Pi-Hole acts as a DNS sinkhole, blocking black-listed domains and forwarding valid ones to the internal Bind9 DNS server acting as upstream DNS server.

Bind9 provides several additional features, like local zones, recursive DNS, or forwarding to public DNS servers.
