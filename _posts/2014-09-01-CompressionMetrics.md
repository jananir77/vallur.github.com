---
layout: post
title: Compression - what is the right API to use?
category: Coding
tags: java Compression SNAPI LZ4 GZIP metrics ZLIB 
year: 2014
month: 09
day: 2
published: true
summary: Compression is used by a system to support Sending large packets over network faster. When the communication link is slower the data can be sent faster as we will be sending less number of bytes. Compression would be generally advantageous if request or response size is more than 1000 bytes, to reduce time spent on the network. 

image: uncompresstime1.png
---

Compression is used by a system to support many of the below advantages

* Send large packets over network faster.
    * When the communication link is slower the data can be sent faster as we will be sending less number of bytes. 
    * Compression would be generally advantageous if request or response size is more than 1000 bytes, to reduce time spent on the network. 
* When we are constrained by space data can be compressed and stored.
* compression would also help while storing to a slow media.
	* We typicaly use SSD or HDD for storing data for long term storage which are 72 to 150+ times slower than temporary storage RAM. This is the reason why most high transactional systems are more memory bound.

Trying to compress and uncompress large number of bytes is counter productive so we would try to find a high bar and stick with that to understand the boundaries.

There are two flavors of famous compression algorithms

Huffman (Tree Based) - Inflate/Defelate, GZip 
Lempel-ziv compression (Dictionary based) - LZ4, LZF, Snappy  

Now for some metrics on the different code that can be used for compression and de-compression using different API. In the below examples we will be only looking at the fastest compression/de-compression with the different API available with their speed and size of compression.

GZip code -
<script src="https://gist.github.com/vallur/dd9c25923548d0c3b4d1.js"></script>

Snappy code -
<script src="https://gist.github.com/vallur/0c0c552fd3e7f650570d.js"></script>

LZF - ning compress code - 
<script src="https://gist.github.com/vallur/b693eb7fa1cb84b71995.js"></script>

Deflator/Inflator code - 
<script src="https://gist.github.com/vallur/fa2b95100a51afa856d1.js"></script>

LZ4 compression
<script src="https://gist.github.com/vallur/a39ff248e98be4c1146c.js"></script>

common code used by all the above API's -
<script src="https://gist.github.com/vallur/6b33e8f2db1bbbf3f26a.js"></script>
  
Let us see different performance and compression metrics of the different flavors listed above and their advantages
 
Below graph shows the size of compression for different compression algorithms. 

 ![compression size](/img/posts/compresssize.png)

Below graph shows the time taken in underseconds but displayed in milliseconds for un-compressing bytes based on the size 

 ![un-compression size](/img/posts/uncompresstime.png)

Below graph shows the time taken in underseconds but displayed in milliseconds for compression based on the size

 ![compression size](/img/posts/compressiontime.png)
 
My experience on compression API is all the implementations that are stream based are too slow as they will have to use frequent allocation of memory bytes which results in frequent movement of bytes to resize arrays 
hence too much wastage of memory as well as processing. It is always better to preallocate in the begining and then shrink the buffer if needed resulting in speed of processing. 

total compression and uncompression time should be close to O(k) + O(n) + O(t)

- k<n and t<n
- k is total size of compressed buffer
- n is total size of uncompressed buffer
- t is the total size of dictionary or repetitive values

Compression using huffman algorithm is slow as it has to build the tree structure that represents the total buffer.

Based on the above metrics 
The order of preference for compression based on size would be 

1. GZIP
2. Deflate/Inflate
3. Snappy
4. LZF
5. LZ4

The order of preference for time taken for compression/decompression would be 

1. LZ4
2. Snappy
3. Deflate/Inflate
4. LZF
5. GZIP

LZ4 is the clear winner when it comes to speed and next winner is deflate/inflate API when saving space.

We need to be careful while choosing Deflate/Inflate as we might not get the bang for while using a slow server, we may spend too much time on compression rather than network time if we use deflate/inflate compressions. Banyan can be configured to use LZ4, Snappy, Deflate/Inflate algorithms.

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
