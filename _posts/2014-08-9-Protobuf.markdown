---
layout: post
title: Java Object Binary Serialization Metrics using ProtoBuf
category: Coding
tags: java Serialization protobuf
year: 2014
month: 08
day: 8
published: true
summary: Serialization is a very big part of modern java applications. Mostly simplicity and readability scores  over performance. If we have a standard format that is small enough and understandable by everyone we would hop on to pick up the winner.

image: protobuf.png
---
Serialization is a very big part of modern java applications. Mostly simplicity and readability scores  over performance. If we have a standard format that is small enough and understandable by everyone we would hop on to pick up the winner.

In this blog i shall look at implementing classes from the serialization point of view and try to serialize and deserialize it and try to find the performance of the same using our friendly TimeLapseCounter. I will be using this serialization for the sake of replication, transaction logging and disk storage.

Serializing and DeSerializing java objects using Protobuf. Protobuf actually kicks java serialization's back when it comes to performance. I would like to decide that this would be my choice for my application. But it has its own limitations as protobuf does not have proper support for Maps and classes with templates or generics.

Some of the reasons why protobuf is extremely fast is listed below

- The byte array size needed to serialize an object is calulated and allocated first rather than on the fly. The way array resizing is done it is faily expensive operation. This is one of the places where protobuf stands way ahead of the curve.

- All fields are marked with a number and set as required or optional to make the expectations on field location in a stream well known. By doing that they make their code generation as simple as 

<script src="https://gist.github.com/vallur/828186685d776abbad7e.js"></script>

- When dealing with Strings they use the concept of lazy string. While deserializing string values they are deserialized and stored as bytes and actually deserialized to String only when they are accessed via their getter method. This i think is clever as the actual time taken is moved to else where.

I would say google has successfully written a code generator that works exactly like how we would write a externalize function to serialize and deserialize our objects.

I created a not so complex ComplexItemContainer schema in protobuf

```
package com.banyan.protobuf;

option java_package = "com.banyan.protobuf";
option java_outer_classname = "ComplexItemContainer";

message ComplexItemList
{
 repeated ComplexItem item = 1;
}

message ComplexItem
{
 required string name = 100;
 repeated string value = 101;
 optional int32 intvalue = 102;
 optional int64 longValue = 103;
 optional int64 doubleValue = 104;
}
```
The way protobuf implements the serialization is by maintaining a map of all the known types listed above for a class and serializing/deserializing the class based on the Map. Run the protoc command with the .proto file to generate the class file. You can download the protoc.exe file from here. In mac you will have to download the protobuf compile code run

```
./configure
./make
./make install
```
to install probuf compiler. This is the command i used to generate java source from protoc compiler

```
protoc -I=d:\school\JMXCounters --java_out=d:\school\JMXCounters\src D:\school\JMXCounters\ComplexItemContainer.proto 
```

Below is the code i used for performing metrics calculation.

<script src="https://gist.github.com/vallur/acd4469c10d92a86f43b.js"></script>

The actual byte size used for serialization in first pass is 120k. The algorithm i used above is very generic and has least amount of repetition of string or int values possible. My idea is to get the worst case performance for a object like this. My performance counters helped in getting the metrics below.

![Protobuf Serialize](/img/posts/protobuf1.png)
![Protobuf deSerialize](/img/posts/protobuf2.png)

When I reduce the LOOPCOUNT to 1 and keep the data size to 100 bytes the time taken in serialization is less than 10 Us cannot get any better.

Ironically serialization is faster than deserialization. But i don't care about that as long as we are still in the low Us ranges as i am not using a powerful machine and we will have enough buffer for network unknowns. Java native serialization is 10x worse than protobuf so i will not show that and you can do your own metrics calculations.

To summarize only drawback is not having the support for Map and classes with generics support. if you know the datatype and class structure protobuf is the one for you. Due to this limitation i will have to look further for other serialization frameworks or implement my own the search continues..... 

<div class="row">	
    <div class="span9 columns">    
		<h2>Comments Section</h2>
	    <p>Feel free to comment on the post but keep it clean and on topic.</p>	
		<div id="disqus_thread"></div>
		<script type="text/javascript">
			/* * * CONFIGURATION VARIABLES: EDIT BEFORE PASTING INTO YOUR WEBPAGE * * */
			var disqus_shortname = 'vallur'; // required: replace example with your forum shortname
			var disqus_identifier = '{{ page.url }}';
			var disqus_url = 'http://vallur.github.io{{ page.url }}';
			
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
