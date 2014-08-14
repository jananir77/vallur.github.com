---
layout: post
title: Serialization - Continues....
category: Coding
tags: java serialization externalize
year: 2014
month: 08
day: 10
published: true
summary: To continue further FST is not as fast in serializaton and deserialization as protobuf. It is very effective and i think i can work on top of both these ideas and develop something better that proper testing can tell.

image: Serialization.png
---

As a continuation of the previous serialization blogs I am planning to bore further on this topic. I like protobuf a lot and also I like FST serialization on the aspects that it can compact the serialized bytes in o(1). Due to the compaction of size between protobuf and FST I would say FST would win if we look at the end to end pipe line.

After learning the best from both these ideas i planned to write my own API and see if i can do better. What is that i have to offer that is not already in one of these. I am planning to add the below features on top of FST framework.

1. The use of ByteBuffer for allocating memory for the objects to be serialized.
	This is particularly important as this widens the range for serialization to use 
	
	DirectByteBuffer - though slow right now might become super fast sooner as OS handles the memory allocation for later. Today it is slow as the JVM controls the availability of the buffer memory for security and exclusive access.
	
	Array backed byte buffer - This is the normal bytebuffer. This is the fastest byte buffer implementation to use.
	
	Memory mapped byte buffer - This is the coolest feature for writing the serialized data asynchronously to disk.
	
2. Pre-calculating the size of a object, so we donot incur the cost of array resizing on the fly. We would incur the array resizing cost and GC cost if we use FST or ByteArrayOutputStream. It is possible to use same bytebuffer to serialize multiple objects that are part of the same connection as long as the size matches.

3. Introduction of LazyString. This is a boxed type of string to store the bytes of string and do the conversion of string to bytes and vice versa only when it is necessary.

4. Only works with known types primitive types and their respective boxed objects, List, Map and some more objects specific to banyan. This looks custom built for banyan but is very easy to develop.


<div style="height:330px;overflow-y:scroll">
<script src="https://gist.github.com/vallur/ba2856722600cc7a6653.js"></script>
</div>
	
After explaining the benefits of the new API. It is time to show the code that does the actual serialization and deserialization for Banyan and check out their performance.

There are two use cases of objects i tend to use in Banyan commonly they are ArrayList for storing single items attribute or leaf values or a Map to store the values that can live in a branch which can protentially be deep.

In this blog i shall work with only ArrayList and my own RandomItem. Its schema looks like below with getter and setters.

<div style="height:330px;overflow-y:scroll">
<script src="https://gist.github.com/vallur/d1e33458708115d817c8.js"></script>
</div>

Now for the code fragment used for the perf test of the new BinaryObjectSerializer

<div style="height:330px;overflow-y:scroll">
<script src="https://gist.github.com/vallur/ffc0a293afb670973e9a.js"></script>
</div>

Now for the actual results and data size after serialization for a total call count of 10,000,000.

The list size of 1000 count the metrics are below

API    |  Byte Size  |	Performance AVG | 
 ---------- | ----------- |	--------------- | 
 Banyan | 172582 | 1 ms 500 Us | 
 [Protobuf](https://code.google.com/p/protobuf/) | 247392 | 6 ms |
 [Fast](http://ruedigermoeller.github.io/fast-serialization/) | 203696 | 21 ms |
 Java | 342209 | 45 ms |

For a list size of 750 count the metrics are below 

API    |  Byte Size  |	Performance AVG | 
 ---------- | ----------- |	--------------- | 
 Banyan | 129717 | 954 Us | 
 [Protobuf](https://code.google.com/p/protobuf/) | 185766 | 4 ms |
 [Fast](http://ruedigermoeller.github.io/fast-serialization/) | 152995 | 14 ms |
 
 For a list size of 500 count the metrics are below 

API    |  Byte Size  |	Performance AVG | 
 ---------- | ----------- |	--------------- | 
 Banyan | 85737 | 544 Us | 
 [Protobuf](https://code.google.com/p/protobuf/) | 124262 | 3 ms |
 [Fast](http://ruedigermoeller.github.io/fast-serialization/) | 101265 | 9 ms |
 
  For a list size of 250 count the metrics are below 

API    |  Byte Size  |	Performance AVG | 
 ---------- | ----------- |	--------------- | 
 Banyan | 43397 | 474 Us | 
 [Protobuf](https://code.google.com/p/protobuf/) | 61798 | 1 ms |
 [Fast](http://ruedigermoeller.github.io/fast-serialization/) | 51175 | 5 ms |
 
 Now going and looking at the more realistic use cases
 
   For a list size of 100 count the metrics are below 

API    |  Byte Size  |	Performance AVG | 
 ---------- | ----------- |	--------------- | 
 Banyan | 17711 | 153 Us | 
 [Protobuf](https://code.google.com/p/protobuf/) | 24260 | 750 Us |
 [Fast](http://ruedigermoeller.github.io/fast-serialization/) | 20839 | 2 ms |
 
    For a list size of 25 count the metrics are below 

API    |  Byte Size  |	Performance AVG | 
 ---------- | ----------- |	--------------- | 
 Banyan | 4301 | 25 Us | 
 [Protobuf](https://code.google.com/p/protobuf/) | 6079 | 75 Us |
 [Fast](http://ruedigermoeller.github.io/fast-serialization/) | 5104 | 250 Us |
 
     For a list size of 1 count the metrics are below 

API    |  Byte Size  |	Performance AVG | 
 ---------- | ----------- |	--------------- | 
 Banyan | 174 | 5 Us | 
 [Protobuf](https://code.google.com/p/protobuf/) | 218 | 10 Us |
 [Fast](http://ruedigermoeller.github.io/fast-serialization/) | 263 | 14 Us |
 
 Now i just need to have support for serializing map types with these super great performance numbers - i get to jump to my next great solution. I am super excited on seeing these metrics. I would be willing to make this genenral purpose if anyone is interested in getting more about this. Based on these metrics I decided to write my own Serialization API's as listed above and they will be avialbale in the banyan repository soon. 
 
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
