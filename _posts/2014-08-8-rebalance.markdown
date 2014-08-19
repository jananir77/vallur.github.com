---
layout: post
title: Rebalancing data across different shards
category: Coding
tags: java, Rebalancing, JVM
year: 2014
month: 08
day: 8
published: true
summary:  This is a continuation of the post how to split keys evenly across different buckets/machines. In this we will be looking at how to rebalance the buckets when we want to increase a bucket size as this operation is a little trickier than simple array copy.
image: java.jpg
---
This is a continuation of the post how to split keys evenly across different buckets/machines.

In this we will be looking at how to rebalance the buckets when we want to increase a bucket size as this operation is a little trickier than simple array copy.

I had outlined the code below.

<script src="https://gist.github.com/vallur/11e497d748f41d3783a1.js"></script>

There are 4 steps involved in rebalancing

1. Set the rebalancing in progress flag to true so the get operation can look for a key in the new bucket when not available on the old one.
2. Set the bucket size value to the new bucket size. We maintain a variable m_oldBucketSize for the get operation fallback logic.
3. Loop through the iterator and copy all the elements that need to go to new bucket.
4. Loop through the iterator and remove all elements from old bucket.

Steps 3 and 4 are done separately to provide consistent responses to get and put operations.

There is a slight change in the get operation compared to the previous blog.

<script src="https://gist.github.com/vallur/6f720f275595e9c3025f.js"></script>

Sample code that shows how the rebalancing would be effective and would work.

<script src="https://gist.github.com/vallur/2688ba81c2b72b1bcee2.js"></script>

###Output:###
<pre></code>
4 - key Belongs to bucket0
1 - key Belongs to bucket1
5 - key Belongs to bucket1
2 - key Belongs to bucket2
3 - key Belongs to bucket3
result after rebalance
5 - key Belongs to Bucket0
1 - key Belongs to bucket1
2 - key Belongs to bucket2
3 - key Belongs to bucket3
4 - key Belongs to Bucket4
</code></pre>

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
