---
layout: post
title: Compression - what are the right places to use them?
category: Coding
tags: java Compression SNAPI LZ4 GZIP metrics ZLIB 
year: 2014
month: 09
day: 2
published: false
summary: Most of the pitfals of in-memory solutions is that they become expensive very fast as we try to store evrything in memory and keep 4 copies for redundency. Also modern applications dont behave properly while we try to use a huge heap size. All of the above would need to make a system elastically scalable and in the end unusable. 

image: clientserver.png
---

We think about compression to reduce storage space and increase the speed of sending packets over the network. On developing a fully inmemory solution I am constrained by only supporting data that can fit inside memory which would be in todays world anywhere from 12 to 24GB if we are using a comodity hardware. 

There are a few ways to solve this problem and support more than what can fit in memory by using one of the below techniques.

1. Use memory mapped buffers to read and write to disk (have the OS do all the paging for us using virtual memory). - Not exactly fully in memory.
2. Allocate more memory to the JVM than what the server has which would be easier than above and the OS and JVM will hanfle the paging without the use of memory mapped buffer.
3. Compress data that is not used often and store them as compressed bytes in memory. 

By logically grouping data together and compressing grouped data that can be accessed together we will be able to keep more data in memory. The key while taking this approach is to use a un-compression algorithm which would take less than acceptable threshold for both compress and uncompress operation ex: 1 to 10 ms and also to choose a minimal data bucket size so that the impact of compression would be low for servicing requests. The metrics I provide below will help us with that. 

This approach would be easily used with the [memory efficient ArrayMap](http://https://github.com/vallur/banyan/tree/master/serialization/src/com/banyan/data/map) implementation provided before. 



By using 1,3 or 2,3 from above i have made tremendous improvements in using 

Instead of turning the focus directly towards compression. I am going to see the most disadvantages of modern day distributed databases and how to mitigate these problems.

* Range queries -
 
If all data lives in memory and the dataset is in terra to peta bytes over commodity hardware our range queries will have to hit 100's if not 1000's of servers before bringing back a response. If our system has lots of these queries then we will have to maintain a army of routers to service these requests.

* Big cluster -

Data can go out of proportion which could end up in having a huge cluster. We all know that commodity hardwares are cheaper and if we buy 10 servers one server can defenitely go bad. In which case if we have 1000 servers 100 servers could go bad which results to chaos.

* Wait I want a portion of my data also in a different datacenter - 

It would not be possible to replicate only a portion of my data to another data center. The only way to do something like this is by creating a seperate database for data like this.

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
