---
layout: post
title: Frequency, Sum, Average and Map Reduce with java 8
category: Coding
tags: java, Agregation, java 8
year: 2014
month: 08
day: 7
published: true
summary: How can do group by and get the count of occurrences of objects in a list. We want to do that based on certain attributes only. 

image: java.png
---
Problem
---
How can we do group by and get the count of occurrences of objects in a list. We want to do that based on certain attributes only.

The Object we will be using for holding the value will be Product

<script src="https://gist.github.com/vallur/666480f53ee40dfa1745.js"></script>

We will be using an ArrayList to hold a collection of products. I would like to know the number of products being sold for each currency. 

Solution
---
<script src="https://gist.github.com/vallur/30fa4bb238d42014a7b3.js"></script>

If you want to do the same code using stream in java 8 the below is the way to go. But good luck debugging through this. The map reduce like api's introduced in java 8 are really cool but is something i have been using in C#.net since 3.5. Both have the same problem.

<script src="https://gist.github.com/vallur/1493f88b5c8caf2c27cb.js"></script>

comparing performance of both the above implementation the stream implementation is slightly lower in performance when the list size grows higher but can be easily parallelized. 

If you want to get a sum of all the product quantity grouped by currency then the below code will work

<script src="https://gist.github.com/vallur/d38b56f4ea6c78645ee9.js"></script>

If you want to get the average of the product quantity based on Currency the below code will do.

<script src="https://gist.github.com/vallur/aa048084971f637d646f.js"></script>

Now how can we get the list filtered based on currency "USD".

<script src="https://gist.github.com/vallur/9ce7ee9e9bca9b492723.js"></script>

The map reduce based aggregation operations in java is very powerful. Until now we have only seen the simplicity at which we can write map reduce like operations using java 8 this is all pretty code to me and saving the number of lines to write. Performance is the same either you use the Aggregation API or do it your self using plain HashMap or Arrays. 

When we can do any of the above operations in parallel using all the available CPU's for a single map-reduce then the power shows up. Let us take the avg example and try to do it with parallel stream

<script src="https://gist.github.com/vallur/f5d1d5aa0c4fcad2bb9f.js"></script>

Yup plain and simple just adding parallelStream gives the solution. Please drop a comment if you like this blog.
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
