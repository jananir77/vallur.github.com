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

image: jconsolenotification.png
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
    <div class="span9 columns">    
		<h2>Comments Section</h2>
	    <p>Feel free to comment on the post but keep it clean and on topic.</p>	
		<div id="disqus_thread"></div>
		<script type="text/javascript">
			/* * * CONFIGURATION VARIABLES: EDIT BEFORE PASTING INTO YOUR WEBPAGE * * */
			var disqus_shortname = 'vallur'; // required: replace example with your forum shortname
			var disqus_identifier = '{{ page.url }}';
			var disqus_url = 'http://erjjones.github.com{{ page.url }}';
			
			/* * * DON'T EDIT BELOW THIS LINE * * */
			(function() {
				var dsq = document.createElement('script'); dsq.type = 'text/javascript'; dsq.async = true;
				dsq.src = 'http://' + disqus_shortname + '.disqus.com/embed.js';
				(document.getElementsByTagName('head')[0] || document.getElementsByTagName('body')[0]).appendChild(dsq);
			})();
		</script>
		<noscript>Please enable JavaScript to view the <a href="http://disqus.com/?ref_noscript">comments powered by Disqus.</a></noscript>
		<a href="http://disqus.com" class="dsq-brlink">blog comments powered by <span class="logo-disqus">Disqus</span></a>
	</div>
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
