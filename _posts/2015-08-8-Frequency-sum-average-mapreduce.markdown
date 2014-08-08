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

image: java.jpg
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
