---
layout: post
title: JMX Notification Listener
category: Coding
tags: java, JMX, Monitoring, JVM
year: 2014
month: 08
day: 8
published: true
summary: The Java notification framework is mainly used in the cache eviction space. It is a reliable way to determine wether a particular server is active and healthy as it would be constantly sending notifications and performance metrics using the JMX API.

image: jconsoleNotification.png
---
The Java notification framework is mainly used in the cache eviction space. It is a reliable way to determine wether a particular server is active and healthy as it would be constantly sending notifications and performance metrics using the JMX API.

The notification framework works in the publisher subscriber model. In this example we can see how the NotificationListener can be used to subscribe to messages from a JMX client.

The below code

<script src="https://gist.github.com/vallur/764ae024e101d7cc5bfe.js"></script>

The above is a simple example to print all the object names that are available in the application that runs and sends its JMX notifications on port 9999.

```
Create an RMI connector client and connect it to the RMI connector server

MBeanServer default domain = DefaultDomain

MBean count = 27

Query MBeanServer MBeans:
 ObjectName = JMImplementation:type=MBeanServerDelegate
 ObjectName = com.banyan.SimpleThreadPool:type=SimpleThreadPoolCounter
 ObjectName = com.banyan.deserialize:type=TimeLapseCounter
 ObjectName = com.banyan.get:type=TimeLapseCounter
 ObjectName = com.banyan.put:type=TimeLapseCounter
 ObjectName = com.banyan.serialize:type=TimeLapseCounter
 ObjectName = com.sun.management:type=DiagnosticCommand
 ObjectName = com.sun.management:type=HotSpotDiagnostic
 ObjectName = java.lang:type=ClassLoading
 ObjectName = java.lang:type=Compilation
 ObjectName = java.lang:type=GarbageCollector,name=PS MarkSweep
 ObjectName = java.lang:type=GarbageCollector,name=PS Scavenge
 ObjectName = java.lang:type=Memory
 ObjectName = java.lang:type=MemoryManager,name=CodeCacheManager
 ObjectName = java.lang:type=MemoryManager,name=Metaspace Manager
 ObjectName = java.lang:type=MemoryPool,name=Code Cache
 ObjectName = java.lang:type=MemoryPool,name=Compressed Class Space
 ObjectName = java.lang:type=MemoryPool,name=Metaspace
 ObjectName = java.lang:type=MemoryPool,name=PS Eden Space
 ObjectName = java.lang:type=MemoryPool,name=PS Old Gen
 ObjectName = java.lang:type=MemoryPool,name=PS Survivor Space
 ObjectName = java.lang:type=OperatingSystem
 ObjectName = java.lang:type=Runtime
 ObjectName = java.lang:type=Threading
 ObjectName = java.nio:type=BufferPool,name=direct
 ObjectName = java.nio:type=BufferPool,name=mapped
 ObjectName = java.util.logging:type=Logging
```

This one spits out all the objects available and since I am not writing a general purpose JMX client. I want to listen to specific counters for notification. Now the enum that we defined for TimeLapseCounters will help us here as it holds an object for the deserialize method.

We will be adding our class for listening to Notification. This class simply spits out the error message it receives from a notification.

<script src="https://gist.github.com/vallur/c317ff2125a9297603c5.js"></script>

In order to listen to notifications we need to add the below two lines to our MiscNotificationListener class 

<script src="https://gist.github.com/vallur/9d5656495332870e9758.js"></script>

The final output that includes the notification error messages is listed below

<script src="https://gist.github.com/vallur/d8820a29cda0bbcbb496.js"></script>


![JMX Monitoring](/img/posts/notification.png)

The javax.management package can be used in your application to listen to messages and act accordingly to resolve conflicts. Look for a blog on that soon.

<div class="row">	
	<div class="span9 column">
			<p class="pull-right">{% if page.previous.url %} <a href="{{page.previous.url}}" title="Previous Post: {{page.previous.title}}"><i class="icon-chevron-left"></i></a> 	{% endif %}   {% if page.next.url %} 	<a href="{{page.next.url}}" title="Next Post: {{page.next.title}}"><i class="icon-chevron-right"></i></a> 	{% endif %} </p>  
	</div>
</div>

<div class="row">	
    <div class="span9 columns">    
		<h2>Comments Section</h2>
	    <p>Feel free to comment on the post but keep it clean and on topic.</p>	
		<div id="fb-root"></div>
<script>(function(d, s, id) {
  var js, fjs = d.getElementsByTagName(s)[0];
  if (d.getElementById(id)) return;
  js = d.createElement(s); js.id = id;
  js.src = "//connect.facebook.net/en_US/sdk.js#xfbml=1&version=v2.0";
  fjs.parentNode.insertBefore(js, fjs);
}(document, 'script', 'facebook-jssdk'));</script>
<div class="fb-comments" data-href="http://vallur.github.io{{ page.url }}" data-numposts="5" data-width="700" data-colorscheme="light"></div>
</div>
<!-- Twitter -->
<script>!function(d,s,id){var js,fjs=d.getElementsByTagName(s)[0];if(!d.getElementById(id)){js=d.createElement(s);js.id=id;js.src="//platform.twitter.com/widgets.js";fjs.parentNode.insertBefore(js,fjs);}}(document,"script","twitter-wjs");</script>

<!-- Google + -->
<script type="text/javascript">
  (function() {
    var po = document.createElement('script'); po.type = 'text/javascript'; po.async = true;
    po.src = 'https://apis.google.com/js/plusone.js';
    var s = document.getElementsByTagName('script')[0]; s.parentNode.insertBefore(po, s);
  })();
</script>
