---
layout: post
title:  "How to remove obsolete JDKs on MacOSX"
date:   2017-05-16 10:40:17 +0200
categories: java
---

A Java developer usually maintains on his machine a collection of different versions of [Java JDK]s to compile and test his applications.

After some time we discover that we want to remove obsolete JDKs after having added other versions.

Unfortunately, on MacOSX there is no an easy way to do that.

So these are the steps to perform a **clean** removal of an existing JDK.

First of all we would like to know how many JDKs are installed on our machine. To do that we open a terminal and type:

{% highlight bash %}
pkgutil --pkgs|grep com.oracle
{% endhighlight %}

on my machine, as an example, I get:

{% highlight msdos %}
com.oracle.jdk7u79
com.oracle.jdk8u112
com.oracle.jdk8u31
com.oracle.jdk8u65
com.oracle.jdk8u77
com.oracle.jre
{% endhighlight %}

Let's remove version 77 of JDK 8. First of all we need to know where the JDK files are located. To obtain this info:

{% highlight bash %}
pkgutil --info com.oracle.jdk8u77
{% endhighlight %}

I get:

{% highlight msdos %}
package-id: com.oracle.jdk8u77
version: 1.1
volume: /
location: Library/Java/JavaVirtualMachines/jdk1.8.0_77.jdk
install-time: 1459414940
{% endhighlight %}

Very well, now we know that we must remove all files under

{% highlight bash %}
/Library/Java/JavaVirtualMachines/jdk1.8.0_77.jdk
{% endhighlight %}

and we do that with root permissions:

{% highlight bash %}
sudo rm -R /Library/Java/JavaVirtualMachines/jdk1.8.0_77.jdk
{% endhighlight %}

Ok, now we must only remember to delete the package information:

{% highlight bash %}
sudo pkgutil --forget com.oracle.jdk8u77
{% endhighlight %}

To confirm that we effectively delete the package, we can re-issue

{% highlight bash %}
pkgutil --pkgs|grep com.oracle
{% endhighlight %}

Finally, to add another JDK, it is sufficient to download the image from OpenJDK or [Oracle] site, and install it like any other MacOSX package.

(A small Gist with all the needed commands is available [here])

[Java JDK]: http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html
[Oracle]: http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html
[here]: https://gist.github.com/guildenstern70/8730c4baae9b3b671f3d3712c86cf2a6

