---
layout: post
title: Throwable Vs Exception which one to catch?
category: Coding
tags: java, Exception
year: 2014
month: 08
day: 8
published: true
summary: I was writing my web application using tomcat and after some iterative changes and unit test changes my unit test succeeded but my web app hangs on a servlet listener class. I kept waiting for sometime and nothing happened. Was looking for an error all it said was failed for the previous error in the log file. 

image: java.png
---
This is a great discussion topic to either impress your fellow developer or offend him.

I shall be dumping my brain out here to show it matters to catch throwable sometimes compared to Exception. Let us look at the below scenarios.

I was writing my web application using tomcat and after some iterative changes and unit test changes my unit test succeeded but my web app hangs on a servlet listener class. I kept waiting for sometime and nothing happened. Was looking for an error all it said was failed for the previous error in the log file. I debugged through the code and it puked out on the middle of a method in one of the classes and went directly to the finally block. Then i did catch Throwable to log the error before coming out.

The error i got is given below.

```
java.lang.NoSuchMethodError: com.google.common.base.Stopwatch.createStarted()Lcom/google/common/base/Stopwatch;
 at TreeStor.DataStor.deSerialize(DataStor.java:148)
 at TreeStor.DataStor.getInstance(DataStor.java:42)
 at TreeStor.Listener.DeSerializeListener.contextInitialized(DeSerializeListener.java:47)
 at org.apache.catalina.core.StandardContext.listenerStart(StandardContext.java:4760)
 at org.apache.catalina.core.StandardContext.startInternal(StandardContext.java:5184)
```
of course it is my fault I had downloaded 3 different versions of guava API compared to using only the latest. This is the only class i used. I used the Guava API to test benchmark on their hash map implementation with java's as well as mine. Ended up using sun JRE hashMap in the end.

Now I don't need Throwable, right? wrong! I ran the web app again and it failed again with the error below

```
java.lang.NoClassDefFoundError: org/xerial/snappy/SnappyOutputStream
 at TreeStor.DataStor.deSerialize(DataStor.java:154)
 at TreeStor.DataStor.getInstance(DataStor.java:42)
 at TreeStor.Listener.DeSerializeListener.contextInitialized(DeSerializeListener.java:47)
 at org.apache.catalina.core.StandardContext.listenerStart(StandardContext.java:4760)
 at org.apache.catalina.core.StandardContext.startInternal(StandardContext.java:5184)
```
The above two are usually user error or OPS error which can happen when new software is installed or during development. But ironically very hard to troubleshoot when it happens. Imagine if you have a dependency on spring and you forgot to package one of the jar files, then good luck finding which one without catching Throwable. This error happens when you have a class during compile time and not during run time.

If you catch Throwable in one of your sub methods you will have to wrap it with your own exception for the cause chain to be bubbled up.

Next for my most favorite error where I can show some tricks. What do you do when you get
```OutOfMemoryError```.

<script src="https://gist.github.com/vallur/91ac53d1f0abae5566eb.js"></script>

The vm option i choose to force the error is "-Xms1m -Xmx1m". This option only allocates 1 MB for the class to force it to throw this error. As you can see even though the class fails with the below error, it still lets the system continue to do the rest of the operation.

```
java.lang.OutOfMemoryError: Java heap space
 at java.util.Arrays.copyOf(Arrays.java:3204)
 at java.util.Arrays.copyOf(Arrays.java:3175)
 at java.util.ArrayList.grow(ArrayList.java:246)
 at java.util.ArrayList.ensureExplicitCapacity(ArrayList.java:220)
 at java.util.ArrayList.ensureCapacityInternal(ArrayList.java:212)
 at java.util.ArrayList.add(ArrayList.java:443)
 at misc.MiscTest.main(MiscTest.java:18)
 at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
 at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
 at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
 at java.lang.reflect.Method.invoke(Method.java:483)
 at com.intellij.rt.execution.application.AppMain.main(AppMain.java:134)
give a closure for your application or this method
``` 
Usually when we write memory intensive applications, we would want to give the system some time before asking it to retry the logic when the OutOfMemory error occurs. Cache eviction can be done to free some memory while this error occurs and then retry.

While running memory intensive application and using this way of coding use the VM option -XX:+UseConcMarkSweepGC and its counterparts to avoid stop the world GC. I shall blog about the different JVM options at a different time.

To conclude based on what you are developing I would suggest you to write your own Exception handling logic where you will decide if something is error or exception and if you can recover from it  or not. Catch the throwable for logging the exception chain in most cases.

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
