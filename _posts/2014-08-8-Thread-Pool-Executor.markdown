---
layout: post
title: Thread Pool Executor
category: Coding
tags: multi threading, java 8
year: 2014
month: 08
day: 8
published: true
summary: It is all about threads. If we have your handle on thread pool we got it all under control. Everything is about how much you can do at a time.

image: SimpleThreadPool.png
---
This time i am going to put the code first and try to explain later.

<script src="https://gist.github.com/vallur/35da0f1db47d63f3d050.js"></script>

This code is used to instantiate the thread pool and process the pool of threads using the ArrayBlockingQueue. I prefer to use the ArrayBlockingQueue vs LinkedBlockingQueue. Every time an object is removed from the LinkedBlockingQueue you will have to mind the GC. Since this is a thread pool and can have a lot of fast to average threads running in them that means you can have extremely large amount of push and pop operations done over a period of time which is constant pressure on GC and also incur locks. Hence the use of ArrayBlockingQueue with small number of size is better. In ArrayBlockingQueue allocation of new memory for each add is prevented.

If using ArrayBlockingQueue, make sure how fast your thread is and how much you want to keep in queue at a given time and set the size accordingly. Whether you use Linked or Array Blocking queue if your queue is full then the thread pool executer blocks the running thread until the queue gets space to fill again.

If you are concerned about processing only the most recent amount of objects in the queue you can implement the Queue interface with the circular buffer and get away with not incurring the cost of array resize as well. If using circular buffer make sure the indexer you use is synchronized. The best of the three would be ArrayBlockingQueue with the occasional cost incurred for resizing.


You need to be careful while setting the size of a queue. The thread pool will be blocked until it gets space to push new threads inside. If we want the thread pool to run for ever with infinite amount of items in queue don't set the size.

<script src="https://gist.github.com/vallur/90303a23ea2d98af08c3.js"></script>

The above code is a small snippet from my No-Sql solution which uses the thread to load all the files in parallel from disk based on their metadata. The metadata collection maintains the schema of all the tables or entities. The relationship between entities are part of the entities itself and not part of schema which makes querying and aggregation super easy.

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
