---
layout: post
title:  "SSH - Multi-Factor Authentication"
date:   2016-10-03 00:45:10 +0100
---

## Intro

### Motivation

SSH offers a variety of authentication methods varying from simple password auth to MFA (multi-factor authentication).

Going with SSH-key based authentication (what you have) where the private key is being protected by a passphrase (what you know) in order to store the same encrypted may be reconsidered as much more secure compared to simple password based authentication.
But one bigger downside here comes from the fact that there is no passphrase expiration or password policy being forced by the default tooling allowing the user to set a weak passphrase or leave the generated private key unprotected.

In addition to the usage of passphrase protected SSH-keys the device or system where the key is being kept has to be hardened accordingly where MFA may become much more useful if all parts of it can be enforced server side.

Simply put, SSH private keys representing what you have in combination with the protecting passphrase (what you know) should not be considered as MFA mainly because not all parts of it can be enforced server side.

## Alternatives

### SSH-Key pair with server side password

Combining the SSH-key authentication with a password based authentication allows us to enforce any password policy also including password expiration/renewal server side - This way the what you know part becomes more controllable or physical.

On SSH configuration level this can be achieved by adding the following line to the SSH-Server config (/etc/ssh/sshd_config):

{% highlight bash linenos %}
AuthenticationMethods publickey,password
{% endhighlight %}

Once the change has been applied and the user being allowed to login over SSH has a key accepted by the server the SSH login should look very similar to the following example:

{% highlight bash linenos %}
➜  ~ ssh localadmin@my-server -i .ssh/id_rsa
Authenticated with partial success.
localadmin@my-server's password:
my-server:~ localadmin$
{% endhighlight %}

### Server side password with OTP
