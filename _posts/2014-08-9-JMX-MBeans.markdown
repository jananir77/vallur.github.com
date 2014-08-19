---
layout: post
title: Monitoring my Java application using JMX MBeans
category: Coding
tags: java, JMX, Monitoring, JVM
year: 2014
month: 08
day: 8
published: true
summary:  I want to get metrics out of my code. In olden days the best way to do that is by using System.out.println. I want to use the below code and see how fast it takes to run serialization and deserialization along with gzip compression.

image: monitoring.png
---
###Introduction###

I want to get metrics out of my code. In olden days the best way to do that is by using System.out.println. I want to use the below code and see how fast it takes to run serialization and deserialization along with gzip compression.

<script src="https://gist.github.com/vallur/976be1ccf23066e59d87.js"></script>

The above code is messy in lot of ways and particularly not reliable in collecting the metrics. Also this code does not give metrics for each run will get quite a few PMD violations. How do i get rid of all these issues and collect proper metrics for how well my serialization code performs.

The best way to do that is by using JMX MBeans which would expose all the needed metrics to JConsole.

###JMX MBeans###

We define our MBean interface that holds all the attributes to be exposed to JConsole

<script src="https://gist.github.com/vallur/f9392eb6fcdd1aa297f9.js"></script>

In the above all long getter that do not have a matching setters will show a neat graph like below when you double click.

![JMX Monitoring](/img/posts/monitoring.png)

Now for the actual implementation of the counter. 

<script src="https://gist.github.com/vallur/02004e61c73290acbf48.js"></script>

The code above is pretty straight forward. All it has is to do with holding values for the time it takes to process single request and a counter to see the number of requests processed. The other information like total average are all calculated. Now we need to expose this counter for each public method which we want to assess for performance.

I would introduce an enum for exposing the counter for each method like below.

<script src="https://gist.github.com/vallur/b90dd2c075ffa69cc282.js"></script>

All the counters for the methods defined in the enum are preregistered with objectName and set to be used. The object name MBean can be monitored in JConsole or JVisualVM using the fully qualified class / method name. 

Now in order to have the counters set with proper value we include it as part of the product code like the sample given below

<script src="https://gist.github.com/vallur/3bc52fcbbaf3105fdbc8.js"></script>

The key part from the above code that enables monitering is <pre>TimeLapseCounters.serialize.getCounter().setCurrentProcessTimeNS(time);</pre>

Now I can increase the number of threads in my thread pool and have the counters collect the metrics and we can monitor it from JConsole/JVisualVM.

###<u>JVisualVM screen shot</u>###

![JMX Monitoring](/img/posts/miscjmx.png)

Now we can see all the counters exposed by the MBeans. I can monitor the performance of the method. I can clearly see that the code is spending too much time in deserializing even though the bytes to read has reduced significantly. The time taken for deserialization does not help when it comes to performance.

You can find more blogs later that talks about serialization metrics, compression metrics and notification.

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
