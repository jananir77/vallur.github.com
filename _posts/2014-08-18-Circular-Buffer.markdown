---
layout: post
title: Circular Buffer using arrays in java
category: Coding
tags: 
year: 2014
month: 08
day: 18
published: true
summary: Circular Buffer implementation has many important usecases from logging to serialization to use fixed memory size.

image: http://luaview.esi-cit.com/dl_pics/circular_buffer.png
---

I have seen many implementations online for implementing Circular Buffer but haven't found a clean one. To me this is a crucial functionality and has a lot of its uses in software. Hence here goes my code.

<script src="https://gist.github.com/vallur/cfe0f9fd94f99fe8c63e.js"></script>

Now you had seen the code here you can see where this can be used. The most commonly used place is logging where the system wants to allocate only certain amount of memory or maintain one hour worth of log data. If your system supports 1000,000 log entries per minute then you would want your array size to support 60 mins worth of data to be 60,000,000. If we don't use a fixed buffer that can be used continuously for a short span of time we would be putting lot of stress on JVM to allocate and free memory. You will be seeing me using this approach in coding for various other types through the blog. 

60,000,000 million sounds like a lot and it is for a small and average system. It is a good idea to preallocate the sizes than to incur the cost of resizing arrays on the fly makes planning memory and H/W for high performant systems easier and less stress on garbage collector.   

If you like my blog drop a line and I am always happy to learn and share more.


<div class="row">	
	<div class="span9 column">
			<p class="pull-right">{% if page.previous.url %} <a href="{{page.previous.url}}" title="Previous Post: {{page.previous.title}}"><i class="icon-chevron-left"></i></a> 	{% endif %}   {% if page.next.url %} 	<a href="{{page.next.url}}" title="Next Post: {{page.next.title}}"><i class="icon-chevron-right"></i></a> 	{% endif %} </p>  
	</div>
</div>

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
