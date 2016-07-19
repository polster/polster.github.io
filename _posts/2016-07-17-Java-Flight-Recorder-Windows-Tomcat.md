---
layout: post
title:  "Java - Flight Recorder with Tomcat on Windows"
date:   2016-07-17 01:20:10 +0100
---

## Tomcat as Windows Service

The following steps describe how to profile a running Tomcat/Java process where Tomcat has been setup as a Windows service; using Java Flight Recorder and Mission Control (requires JDK 7 or newer).

### Preparation

* Open Tomcat's Windows Service configuration helper: *$TOMCAT_HOME\\bin\\Tomcat8w.exe*
* Switch to the Java tab

![Tomcat8w]({{ site.url }}/images/Tomcat8w-Java-Config.png)

* Append the following lines in order to enable flight recorder:
{% highlight bash %}
-XX:+UnlockCommercialFeatures
-XX:+FlightRecorder
{% endhighlight %}
* Ensure the following lines are present as well in order to be able to connect over Mission Control. Adjust the same as needed:
{% highlight bash %}
-Dcom.sun.management.jmxremote
-Dcom.sun.management.jmxremote.host=localhost
-Dcom.sun.management.jmxremote.port=8999
-Dcom.sun.management.jmxremote.ssl=false
-Dcom.sun.management.jmxremote.authenticate=false
{% endhighlight %}
* Click OK and restart Tomcat

### Profiling
