---
layout: post
title:  "Java - In App version with Maven"
date:   2016-03-26 23:26:17 +0100
---

## Intro

In this post I will show you an easy alternative how to use Maven's resource filter feature
and combine it with some portions of Java code in order to make the build version
available to the app at runtime.

Thinking further when it comes to continuous integration the Maven build version
may be set by the CI Server via the [Maven version plugin](http://www.mojohaus.org/versions-maven-plugin/) where
a unique build number may be part of the same. Or such build number could be set over
an additional Java argument as part of the Maven build command where we would need to
introduce a separate property placeholder within our version property file - this to
be explained more in detail in a follow-up post.

## Basic Setup

* The following code snippet demonstrates how to easily read properties from file over the java.util.ResourceBundle:

{% highlight text %}
  /**
  * Create a new instance.
  *
  * @param bundleName Name of the resource bundle with the version information
  */
  public Version(final String bundleName) {
    final ResourceBundle bundle = ResourceBundle.getBundle(bundleName);

    versionNumber = bundle.getString("version");
    name = bundle.getString("name");
    url = bundle.getString("url");
    try {
        timestamp = DateUtils.parseDate(bundle.getString("timestamp"), DATE_PATTERNS);
    } catch (final ParseException e) {
        logger.warn("Failed to parse the version timestamp: " + e.getMessage());
        logger.debug(e.getMessage(), e);
    }

    logger.info(getFullVersion());
  }
{% endhighlight %}

* Placing a simple properties file located on the class path (e.g. <project>/src/main/resources-filtered/ch/mycompany/myapp/version.properties) containing some key/values where the values can be filtered via maven (placeholders) during the build process will make the same available to the app at runtime:

{% highlight text %}
  name=${pom.name}
  version=${pom.version}
  url=${pom.url}
  timestamp=${timestamp}
{% endhighlight %}

* More in detail, passing the resource bundle locator to the code snippet as seen above during initialization will read the properties:

{% highlight text %}
  /**
  * The application version.
  */
  public class AppVersion extends Version {

    private static final String BUNDLE = "ch/mycompany/myapp/version";

    private static AppVersion instance;

    private AppVersion() {
        super(BUNDLE);
    }

    public static AppVersion getInstance() {
        if (instance == null) {
            instance = new AppVersion();
        }
        return instance;
    }
  }
{% endhighlight %}

* Assuming maven as the build tool, we can add the following properties to the project POM in order to let maven generate a build timestamp that can be consumed as property:

{% highlight text %}
  <properties>

    <!-- timestamp used for build info -->
    <timestamp>${maven.build.timestamp}</timestamp>
    <maven.build.timestamp.format>yyyy-MM-dd HH:mm</maven.build.timestamp.format>

  </properties>
{% endhighlight %}

* After successful maven build completion the properties file may look as follows (sample):
{% highlight text %}
  name=impex-freigabeclient-package
  version=1.2.0-456-RC1
  url=http://mycompany.ch/myapp-package
  timestamp=2016-03-21 05:49
{% endhighlight %}
