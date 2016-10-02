---
layout: post
title:  "SSH - Command line configuration on OS X"
date:   2016-10-02 22:45:10 +0100
---

## Intro

### Motivation

The classic way to enable and configure the remote login feature on a Mac is to open System Preferences > Sharing and then enable the feature.

Looking for a more automated approach the following sections will describe or lets say suggest how to achieve the same over Terminal commands.

### Compatible Systems

All adjustments as described below have been tested against the following OS version:
* OS X 10.11

## Useful commands

| Command | Description |
|---------|-------------|
| systemsetup -setremotelogin on | Enable remote login (aka SSH-Server) |
| dscl . -read /Groups/com.apple.access_ssh | Print the members of the group being allowed to login over SSH (these members correspond to the allowed users under Sharing, but may not be displayed in the GUI) |
| dscl . -read /Groups/com.apple.access_ssh-disabled | In case no error is being printed but the full group information then this means any user is being allowed to login via SSH (corresponds to the All Users option under Sharing) |

## SSH Server configuration via CMD on OS X

### Enabling Remote Login

* Permissions are being controlled over directory services where simply enabling remote login without any other configuration will allow all users to login remotely
* Highly recommended you should limit the allowed users by first enable the access_ssh record:
{% highlight bash linenos %}
dscl . change /Groups/com.apple.access_ssh-disabled RecordName com.apple.access_ssh-disabled com.apple.access_ssh
{% endhighlight %}
* And then add one or more users to the record:
{% highlight bash linenos %}
dscl . append /Groups/com.apple.access_ssh GroupMembership USERNAME
{% endhighlight %}
* Finally, enable the remote login service:
{% highlight bash linenos %}
systemsetup -setremotelogin on
{% endhighlight %}

### Changing the allowed user list

* In case you need to add or remove users being allowed to login over SSH, a restart of the service seems not to be required (will reload automatically)
* Add one more user:
{% highlight bash linenos %}
dscl . append /Groups/com.apple.access_ssh GroupMembership USERNAME2
{% endhighlight %}
* Remove user:
{% highlight bash linenos %}
dscl . -delete /Groups/com.apple.access_ssh GroupMembership USERNAME
{% endhighlight %}
