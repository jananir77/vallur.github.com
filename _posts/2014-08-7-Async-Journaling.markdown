---
layout: post
title: How to do async journaling from your server code 
category: Coding
tags: java, journaling
year: 2014
month: 08
day: 19
published: true
summary: I want to write journal entries or log entries of transactions to disk. I don't want the system to slow down due to logging. I want to have a rolling log and should be able to recover when some thing wrong happens. 

image: java.png
---
Problem
---
I want to write journal entries or log entries of transactions to disk. I don't want the system to slow down due to logging. I want to have a rolling log and should be able to recover when some thing wrong happens. Recovery part is not covered in here.

Solution.
---
Simply use a customized version of log4j which itself supports async logging or use flume. While you choose any of the above technologies please remember the footprint it has for your system and what they are designed for.

You can also use a slight variation of the code i outline below. The below code creates a rolling file with a time stamp. The file has kind of a fixed size of 360 MB, it uses super fast compression (LZF Compress). The writes are written to disk using LinkedBlockingQueue first written to a queue and then later written to memory mapped file in disk. The memory mapped files are outside of JVM heap memory so the file contents might not be held in memory during the time of the writes.

We can use either circular buffer or LinkedBlockingQueue both will give performance in o(1). Let us see the advantages and disadvantages of using circular buffer vs LinkedBlokingQueue.

<u>Circular buffer</u>
---
Advantages
---
No time taken for resizing or new memory allocation. Hence no garbage. This is a huge advantage as you know that the GC won't kick-in to claim any of the unused ones from this.

Since we do not resize or add new entries to a fixed buffer there will be no locks. Locks are made only on the part where we increment the index.

Disadvantage
---
This is a circular buffer hence it has the potential to overwrite existing entires. If we do not let it grow over its size we do not have the concern of overwriting the existing ones.

You will have to allocate a certain amount of static size upfront for this one.

<u>LinkedBlockingQueue</u>
---
Advantages
---
* In normal cases this might not reach the size that is considered the worst case.
* It won't overwrite existing entries.

Disadvantages
---
* Can cause Memory issues.
* This will incur memory allocation for each poll / put entry. Hence there will be a lot of additional work for the GC.
* This will incur a lock since this is a LinkedBlockingQueue on both push as well as pop operations.
* A lock will be made on the thread when the queue is full for inserting new elements.

<script src="https://gist.github.com/vallur/1f4a46f59e312bea8642.js"></script>

Since the journal entries are compressed it probably would support 10 hours worth of journal entries in 1 file. This code might potentially chunck the file and write the full snapshot to disk every 10 hours making the system extremely efficient and reducing the disk lag (latency on writing to disk).

Little bit of bragging here.
---
 The No sql system i wrote uses this logic for journaling,Where it writes the full snapshot to disk when ever the 360 MB mark reaches on journaling. Once the full snapshot is made the Journal entries are used only to help outdated masters come back to life. Transferring a 360 MB or less file between masters is way easier than incurring the multiple reads on the system. When ever a different server needs a new snapshot of information the system is designed to give it the snapshot data and journal to bring the new server to life. It takes my system 10 to 16 seconds to load the Snapshot and journal data from disk in my 3 year old MAC laptop. It takes the system 25 seconds to write the snapshot.
 
Need to make more testing to fine tune, will update if i find any issues. 

If you like my blog drop a line and I am always happy to learn and share more.
<div class="row">	
	<div class="span9 column">
			<p class="pull-right">{% if page.previous.url %} <a href="{{page.previous.url}}" title="Previous Post: {{page.previous.title}}"><i class="icon-chevron-left"></i></a> 	{% endif %}   {% if page.next.url %} 	<a href="{{page.next.url}}" title="Next Post: {{page.next.title}}"><i class="icon-chevron-right"></i></a> 	{% endif %} </p>  
	</div>
</div>

<div class="row">	
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
