---
layout: post
title: Circular Buffer using arrays in java
category: Coding
tags: circular buffer, java
year: 2014
month: 08
day: 18
published: true
summary: Circular Buffer implementation has many important usecases from logging to serialization to use fixed memory size

image: circularbuffer.jpg
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
<!--Google analytics-->
<script>
  (function(i,s,o,g,r,a,m){i['GoogleAnalyticsObject']=r;i[r]=i[r]||function(){
  (i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),
  m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;m.parentNode.insertBefore(a,m)
  })(window,document,'script','//www.google-analytics.com/analytics.js','ga');

  ga('create', 'UA-53607051-1', 'auto');
  ga('send', 'pageview');

</script>
