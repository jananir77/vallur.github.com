---
layout: post
title: How to interrupt a thread in java?
category: Coding
tags: java, Thread, JVM
year: 2014
month: 08
day: 8
published: true
summary:  Just like how the OS allows us to kill a long running task we should have the flexibility in our code to stop long running threads.

image: java.jpg
---
Just like how the OS allows us to kill a long running task we should have the flexibility in our code to stop long running threads.

There are many ways to do this.

By calling the Thread.interrupt method. Including the comments from the interrupt method of the thread. Based on code comments from JDK documentation below is the behavior of the interrupt method. If a thread is blocked in an invocation of the wait, join or sleep method the thread will receive a Interrupted exception to continue to the exception handling block of the thread.

If a thread is blocked in an IO operation. The underlying IO channel will be closed and the thread will be interrupted. Interrupting a not alive thread has no effect.

Now after reading this it looks like the thread will be killed just by calling the interrupt method. But it does not work that way. If we write our code like below there is no way for the process to interrupt the thread.

The thread code below will keep continuing and will ignore the interrupt signal as long as the //do some operation block does not involve handling IO operations. 

<script src="https://gist.github.com/vallur/dbcc5916bb8cf972b67a.js"></script>

When operating with thread it is always safe to use a boolean flag to give the underlying thread or runnable breathing time and graciously have a calm death. Hence the introduction of Simple Thread example given below which shows how to gracefully exit threads.

<script src="https://gist.github.com/vallur/33ffbda211665a2dfa85.js"></script>

In order for the above method to be effective we need to add the below interrupt method to the SimpleThreadPool 

<script src="https://gist.github.com/vallur/960b4101f15e7f989674.js"></script>

Disclaimer: Unless properly developed a java thread will leak and never die until the JVM quits even if the interrupt method is called.

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
