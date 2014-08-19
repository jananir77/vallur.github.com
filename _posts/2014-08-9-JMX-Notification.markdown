---
layout: post
title: JMX Notification for alerts
category: Coding
tags: java, JMX, Monitoring, JVM
year: 2014
month: 08
day: 8
published: true
summary:  With the use of javax.management it is possible to notify other users/ machines about cache eviction or perform alerts for errors or warnings. The notification on MBeans can effectively used for building self healing systems. Where one server can notify others if it is overloaded so that they can take charge.

image: jconsoleNotification.png
---
With the use of javax.management it is possible to notify other users/ machines about cache eviction or perform alerts for errors or warnings.

The notification on MBeans can effectively used for building self healing systems. Where one server can notify others if it is overloaded so that they can take charge.

In this example we will see how the notification can be used to show alerts when a error happens.

The code below throws errors when a random value is hit simulating a error scenario 

<script src="https://gist.github.com/vallur/a8afdfb52c2be8a9b53b.js"></script>

I shall be adding these two methods to TimeLapseCounter explained in the previous blog.

<script src="https://gist.github.com/vallur/85e32b04ebaabdafd203.js"></script>

These methods enable the counter to notify when a error is triggered in deSerialize method. All the errors generated will be collected in the Notification Listener in our case it would be JConsole/JVisualVM. We can extend it further to provide a warning when a critical method goes out of SLA by triggering it in the TimeLapseCounter.

In the above example I am using the AttributeChangeNotification class to notify on errors and warnings. I would ideally like to extend the Notification class and expose the counters. The down side to that approach is to have my implementation class be part of the class path of the NotificationListener which would not solve the purpose. If not in class path we would receive this warning in JConsole.

```
Jul 29, 2014 7:42:02 PM ClientNotifForwarder NotifFetcher.fetchOneNotif
WARNING: Failed to deserialize a notification: java.lang.ClassNotFoundException: com.banyan.jmx.notification.LogErrorNotification (no security manager: RMI class loader disabled)
```

My above implementation would work perfectly in all JMX clients.

This is how the error messages will get displayed in JConsole.

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
