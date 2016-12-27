---
layout: post
title:  "Docker Host - Secured remote management"
date:   2016-12-25 23:26:17 +0100
---

## Motivation

* Enabling the Docker daemon socket on a docker host to accept remote connections may be required in certain situations (e.g. automation, centralized management, integration over the Docker API)
* Especially in production and when dealing with sensitive data, the TCP connection shall be protected by enabling TLS
* This post brings together the minimally required steps to 1. configure the Docker daemon socket for TLS and 2. generate the certificates on a sample basis

## Secured remote management

### Prerequisites

The setup as described below has been tested on behalf of the following environment:

* Docker Host based on CentOS/RHL 7
* Docker Engine 1.12.5 (or higher) installed

### SSL Certificates

* Create the needed SSL server and client certificates (best practice here is to go with certificates being signed by an official certificate authority)
* Just for reference and for development usage, here a sample:
<script src="https://gist.github.com/polster/5ea79927cf9a496c7da84ee70d8abdd4.js"></script>
* Keep in mind that best practice is to generate the private key(s) on the machine where they are required, only having to share the request for signing with your CA of choice
* Ensure the needed private key and certificates are being placed securely on the Docker host:
{% highlight bash linenos %}
mkdir /root/.docker
cp server-cert.pem server-key.pem ca.pem /root/.docker
chmod -v 0400 /root/.docker/server-key.pem
chmod -v 0444 /root/.docker/{server-cert.pem,ca.pem}
{% endhighlight %}

### Docker socket config

* Create the following dir:
{% highlight bash linenos %}
mkdir /etc/systemd/system/docker.service.d
{% endhighlight %}
* Create the following service config file:
{% highlight bash linenos %}
vi /etc/systemd/system/docker.service.d/docker.conf
{% endhighlight %}
* Copy paste the following content (adjust as needed):
{% highlight bash linenos %}
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd --tls=true --tlscert=/root/.docker/server-cert.pem --tlskey=/root/.docker/server-key.pem --tlscacert=/root/.docker/ca.pem -H tcp://docker-node01.example.com:2376
{% endhighlight %}
* Reload configuration:
{% highlight bash linenos %}
systemctl daemon-reload
{% endhighlight %}
* Restart the Docker daemon:
{% highlight bash linenos %}
systemctl restart docker
{% endhighlight %}

### Connection test

* Run the following command in order to test the connection over TLS:
{% highlight bash linenos %}
docker --tlsverify --tlscacert=ca.pem --tlscert=cert.pem --tlskey=key.pem -H=docker-node01.example.com:2376 version
{% endhighlight %}

## Appendix

### References

* [Docker - Configure Docker Daemon socket on CentOS 7](https://docs.docker.com/engine/admin/#/configuring-docker-1)
* [Docker - Protect the Docker daemon socket](https://docs.docker.com/engine/security/https/)
