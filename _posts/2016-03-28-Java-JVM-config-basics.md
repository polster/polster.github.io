---
layout: post
title:  "Java - JVM Config Basics"
date:   2016-03-28 01:25:10 +0100
---

## Usage of Java Version depending JVM parameters

### Java 8 and higher

{% highlight text %}
-server
-Xms<heap size>[g|m|k] -Xmx<heap size>[g|m|k]
-XX:MaxMetaspaceSize=<metaspace size>[g|m|k]
-Xmn<young size>[g|m|k]
-XX:SurvivorRatio=<ratio>
-XX:+UseConcMarkSweepGC -XX:+CMSParallelRemarkEnabled
-XX:+UseCMSInitiatingOccupancyOnly -XX:CMSInitiatingOccupancyFraction=<percent>
-XX:+ScavengeBeforeFullGC -XX:+CMSScavengeBeforeRemark
-XX:+PrintGCDateStamps -verbose:gc -XX:+PrintGCDetails -Xloggc:"<path to log>"
-XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=10 -XX:GCLogFileSize=100M
-Dsun.net.inetaddr.ttl=<TTL in seconds>
-XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=<path to dump>`date`.hprof
-Djava.rmi.server.hostname=<external IP>
-Dcom.sun.management.jmxremote.port=<port>
-Dcom.sun.management.jmxremote.authenticate=false
-Dcom.sun.management.jmxremote.ssl=false
{% endhighlight %}

### Before Java 8

{% highlight text %}
-server
-Xms<heap size>[g|m|k] -Xmx<heap size>[g|m|k]
-XX:PermSize=<perm gen size>[g|m|k] -XX:MaxPermSize=<perm gen size>[g|m|k]
-Xmn<young size>[g|m|k]
-XX:SurvivorRatio=<ratio>
-XX:+UseConcMarkSweepGC -XX:+CMSParallelRemarkEnabled
-XX:+UseCMSInitiatingOccupancyOnly -XX:CMSInitiatingOccupancyFraction=<percent>
-XX:+ScavengeBeforeFullGC -XX:+CMSScavengeBeforeRemark
-XX:+PrintGCDateStamps -verbose:gc -XX:+PrintGCDetails -Xloggc:"<path to log>"
-XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=10 -XX:GCLogFileSize=100M
-Dsun.net.inetaddr.ttl=<TTL in seconds>
-XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=<path to dump>`date`.hprof
-Djava.rmi.server.hostname=<external IP>
-Dcom.sun.management.jmxremote.port=<port>
-Dcom.sun.management.jmxremote.authenticate=false
-Dcom.sun.management.jmxremote.ssl=false
{% endhighlight %}

## Best Practices

### Explicit Heap

* Configure the same value for both Xms and Xmx:
{% highlight text %}
  -Xms<heap size>[g|m|k] -Xmx<heap size>[g|m|k]
{% endhighlight %}
* Meta space in Java 8 is unlimited by default, where multiple Java processes would concurrence each other. Therefore we should declare the meta space size explicit:
{% highlight text %}
  -Xmn<young size>[g|m|k]
{% endhighlight %}

### Enable automatic heap dumping on OutOfMemory

* Append the following to your $JAVA_OPTS:
{% highlight text %}
  -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/<path>/dump-$(date +"%Y%m%d%H%M%S").hprof
{% endhighlight %}

### Performance Analysis over GC Logging

* Append the following arguments to you $JAVA_OTPS (recommended, may be adjusted):
{% highlight text %}
  -XX:+PrintTenuringDistribution -XX:+PrintGCCause -XX:+PrintGCApplicationStoppedTime -verbose:gc -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -Xloggc:/opt/<path>/gc-$(date +"%Y%m%d%H%M%S").log
{% endhighlight %}
