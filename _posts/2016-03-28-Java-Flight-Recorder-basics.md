---
layout: post
title:  "Java - Flight Recorder Basics"
date:   2016-03-28 01:20:10 +0100
---

## Basic Setup

* Append the following arguments to your $JAVA_OPTS. Keep in mind without these arguments the flight recorder feature will remain disabled:
{% highlight bash %}
  XX:+UnlockCommercialFeatures -XX:+FlightRecorder
{% endhighlight %}

## Recording by Hand

{% highlight bash %}
# become the user owning the java process
sudo su tomcat

# print the currently running java processes
/usr/java/jdk1.8.0_73/bin/jcmd

# start recording
/usr/java/jdk1.8.0_73/bin/jcmd <proc id> JFR.start name=flight settings=default

# dump recording to file
/usr/java/jdk1.8.0_73/bin/jcmd <proc id> JFR.dump name=flight filename=/opt/tomcat-singapore/logs/flight-20160209.jfr
{% endhighlight %}

## Automatic Recording

* Append the following parameters to your JAVA_OPTS in order to dump the current profiling data to file and start with a new recording on Java process restart (settings may be omitted if no custom profiling config file is present):
{% highlight bash %}
  export JAVA_OPTS="$JAVA_OPTS -XX:StartFlightRecording=name=<flight name>,filename=/<path>/<dump>-$(date +"%Y%m%d%H%M%S").jfr,settings=/<path>/<settings>.jfc,dumponexit=true"
{% endhighlight %}
