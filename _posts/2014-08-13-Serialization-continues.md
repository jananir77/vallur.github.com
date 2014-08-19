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

5. Automatically provides compression and the compression acheived is better than LZ4 compression on top of java serialized bytes.


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
 Banyan | 111,965 | 4 ms | 
 [Protobuf](https://code.google.com/p/protobuf/) | 247,981 | 4 ms |
 [Fast](http://ruedigermoeller.github.io/fast-serialization/) | 120,490 | 7 ms |
 Java | 204,135 | 45 ms |

For a list size of 750 count the metrics are below 

API    |  Byte Size  |	Performance AVG | 
 ---------- | ----------- |	--------------- | 
 Banyan | 78,333 | 4 ms | 
 [Protobuf](https://code.google.com/p/protobuf/) | 184,404 | 3 ms |
 [Fast](http://ruedigermoeller.github.io/fast-serialization/) | 84,467 | 5 ms |
 
 For a list size of 500 count the metrics are below 

API    |  Byte Size  |	Performance AVG | 
 ---------- | ----------- |	--------------- | 
 Banyan | 52,401 | 2 ms | 
 [Protobuf](https://code.google.com/p/protobuf/) | 123,468 | 2 ms |
 [Fast](http://ruedigermoeller.github.io/fast-serialization/) | 56,537 | 2 ms |
 
  For a list size of 250 count the metrics are below 

API    |  Byte Size  |	Performance AVG | 
 ---------- | ----------- |	--------------- | 
 Banyan | 26,288 | 1 ms | 
 [Protobuf](https://code.google.com/p/protobuf/) | 61,471 | 1 ms |
 [Fast](http://ruedigermoeller.github.io/fast-serialization/) | 28,211 | 1 ms |
 
 Now going and looking at the more realistic use cases
 
   For a list size of 100 count the metrics are below 

API    |  Byte Size  |	Performance AVG | 
 ---------- | ----------- |	--------------- | 
 Banyan | 104,40 | 550 µs | 
 [Protobuf](https://code.google.com/p/protobuf/) | 248,84 | 450 µs |
 [Fast](http://ruedigermoeller.github.io/fast-serialization/) | 112,44 | 1 ms |
 
    For a list size of 25 count the metrics are below 

API    |  Byte Size  |	Performance AVG | 
 ---------- | ----------- |	--------------- | 
 Banyan | 25,67 | 120 µs | 
 [Protobuf](https://code.google.com/p/protobuf/) | 62,03 | 100 µs |
 [Fast](http://ruedigermoeller.github.io/fast-serialization/) | 27,98 | 154 µs |
 
     For a list size of 1 count the metrics are below 

API    |  Byte Size  |	Performance AVG | 
 ---------- | ----------- |	--------------- | 
 Banyan | 103 | 7 µs | 
 [Protobuf](https://code.google.com/p/protobuf/) | 238 | 7 µs |
 [Fast](http://ruedigermoeller.github.io/fast-serialization/) | 140 | 10 µs |
 
As you can see from the metrics above all three are comparable on performance. Protobuf is a little faster as it is more declarative. I shall update the post with more fine tuning to acheive better performance.
 
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
