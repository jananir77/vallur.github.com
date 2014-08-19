---
layout: post
title: How to evenly split data across different machines or buckets?
category: Coding
tags: java, sharding, JVM
year: 2014
month: 08
day: 8
published: true
summary:  i am going to demonstrate how this can be effectively done from code using java. You will not be seeing code specific to communicating to different machines but will demonstrate the same using my own custom implementation of ArrayMap and % (MOD) operator.

image: buckets.png
---
Here i am not going to show how this is done in cassandra or MongoDB. But i am going to demonstrate how this can be effectively done from code using java. You will not be seeing code specific to communicating to different machines but will demonstrate the same using my own custom implementation of ArrayMap and % (MOD) operator.

First we need to understand the concept of hashing in java.

What is hashing?
---
Hashing is a way at which an object's data can be represented by a integer value. Since integers have a max of 231-1 there is potential to have collisions based on the key value assuming the keys can be anything. 

We will be using the java hashCode function to get the hash code of an object as shown in the method below.

<script src="https://gist.github.com/vallur/9585a0a37e333c89c312.js"></script>

This code will provide the bucket to which a key belongs to. In order to distribute to a evenly growing bucket we will be using the below at the storage type.

List<MapEntry<K, V>>[] table = null;

Here the array is holding the different buckets and list is the bucket. Here we are not concerned about collisions as the key that belongs to a bucket will be inserted inside the list.

class ArrayMap initializer
<script src="https://gist.github.com/vallur/219f927c2293f56eb853.js"></script>

Code for performing puts in the array. 

<script src="https://gist.github.com/vallur/01e99a2d357a2bc164c8.js"></script>

If I was writing this as a no-sql solution i would be using a similar hash map on each server to maintain a bucket and list so rebalancing can be done easier.

<script src="https://gist.github.com/vallur/48b68d0149d88726054a.js"></script>

Let us see if the above achieves o(1) for get, put and delete. I will not worry about delete just yet.

Adding new values in this approach is definitely o(1) as we pre create the arrays and there is sometimes a onetime minor cost for initializing the LinkedList. Gets on the other hand on a linked list are o(n) which we need faster than puts. The bucket size that determines the number of servers can be small but the next one on each server we need to keep it as big as possible just to ensure each key  lands in its own bucket and the LinkedList is used as a means for conflict resolution. To achieve the o(1) for gets and puts we will be maintaining a smaller number of servers as buckets and a Big Table inside each server for holding the Key,Value.

Big Table as part of each server need not be n^2 for future growth. It can just be n or n/2 as the size of the array grows it becomes counter productive in memory allocation and usage. Using n/2 array size with a max of 1 best case and 20 worst case LinkedList size gives the best performance. The maximum bucket Size i have experimented with is 172 million and the linked list size having an average of 1 to 10. 

As simple as it seems this has its own complexities in performing range queries or a select * operation. Let us tackle the select * problem first. We can tackle this by providing a custom EntrySet implementation as outlined below

<script src="https://gist.github.com/vallur/b44a337568f506f2abbe.js"></script>

The important method in the above is the nextNode and the forEach. Both of them work in a very similar way. One problem with the above approach is that there are nulls in the bucket which will have to be ignored. Until the buckets are fully filled the loops will have to run through the nulls and ignore them.

In the above the foreach is faster than Iterator. As the foreach uses the Iterator on a LinkedList which works based on node.next where the Iterator uses indexing. Indexing on LinkedList is done by o(n). The list on the bucket should be very small this logic should be noted.

Now this solution works great for get, put and full list. If we have to do range queries there is no possible way to do that. But this system will work effectively for range queries as well as the ranges can be split into buckets and queried on the different buckets in parallel making range queries extremely fast. The result can be merged by the client for display purposes.

Look for a blog link here for how to do rebalance.
Look for a blog link here for how to do range queries.

I am not tackling sorting in this solution as sorting will be client side operation as the keys can be distributed across different machines and there is no guarantee to have them sorted on insert based on the actual key unless the key itself is an integer value. Sorting will be handled by the driver or router.

Since this is a sample code it does not contain the synchronization logic for the list creation. If not done will have unexpected results while accessed from multiple threads.

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
