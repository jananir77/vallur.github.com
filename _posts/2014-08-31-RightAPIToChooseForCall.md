---
layout: post
title: Rest Over JSON or plain old sockets for communication
category: Coding
tags: java REST SOA JSON XML
year: 2014
month: 08
day: 31
published: true
summary: REST over JSON is the 1st choice when we try to expose services to the outside world. Though in recent times Web Sockets are raising up to the challenge to replace REST as rest APIS are too chatty to increase the messages sent over the network. 

image: clientserver.png
---

<b>ReST</b> - <b>R</b>epresentational <b>S</b>tate <b>T</b>ransfer) over JSON (<b>J</b>ava<b>S</b>cript <b>O</b>bject <b>N</b>otation) is the 1st choice when we try to expose services to the outside world. Though in recent times Web Sockets are making developers think as a replacement as REST APIS are too chatty and increase the messages sent over the network. 

The community choose ReST over SOAP as ReST is further light weight and doesnot need to adhere to a specific schema like SOAP. ReST is light weight built on top of the http stateless protocol where it supports the operations of CRUD. In short it is no way different from how we write a servlet or any other webpage, in short just a browser is enough to communicate with ReST. The framework is more fine tuned for services that can be represented in the form of resources ex:- product will be a resource and we would suppot the below 4 operations on product with rest.
 
<b>C</b>reate, <b>R</b>ead, <b>U</b>pdate, <b>D</b>elete operations can be used to access a resource using the html methods POST, GET, PUT and DELETE to do the respective operation.

[MyKong](http://www.mkyong.com/webservices/jax-rs/restfull-java-client-with-java-net-url/ "Http Rest methods")  has a simple example to show how the Get and POST are implemented and accessed with ReST service. 

Some of the advantages of using ReST

* ReST has all the protocol and error codes defined for most of the application needs and can be used. 
* It also has support for binary octect stream so that protobuf serialization can be used with little bit of coding added. 
* Caching can be implemented using most of the general purpose http cache applications
* All exisiting major frameworks using IIS/Tomcat/Jetty support ReST by default.
* It is language independent we can use our browser or CURL as a client to make changes.

Disadvantages of using ReST. I have to be clear when i say disadvantages of ReST, it is mostly the disadvantages of how the servers and the protocol is implemented using the ReSTFUL architecture. 

* HTTP is chatty. for example a simple request can have the below headers in exchange.
* Serialization and deserialization to JSON or XML is expensive for memory as well as performance. 

####HttpRequest####
``` 
GET http://localhost:9998/helloworld HTTP/1.1\r\n
Accept-Encoding: gzip,deflate\r\n
Host: localhost:9998\r\n
Connection: Keep-Alive\r\n
User-Agent: Apache-HttpClient/4.1.1 (java 1.5)\r\n
```

####HttpResponse####
```
HTTP/1.1 200 OK\r\n
Transfer-encoding: chunked\r\n
Content-type: text/json\r\n
Date: Wed, 20 Aug 2014 23:34:17 GMT\r\n
 \r\n
Hello World\r\n
```
A total of 285*2 (548) bytes transferred to and from the service. Considering SOAP the ReSTFul Services are less chatty. I used SOAP UI to get these headers and I used the browser it would result in more byte transfer.

* It is stateless that means there can be only one request/response over a single connection. 
	- This does not allow the client to have connection pooling. This is a severe drawback when we want to have a synchronous connection between server and client to perform multiple operations or transfer multiple requests/response over a single connection. 
	- If a client is sending 1000's of requests there needs to be 1000's of connections which translates to 1000's of short lived threads.
* Post method values are Base64 Encoded values. So the actual data transfered on a post request is increased for padding by  4 * [n/3]. If we send 100 bytes for update or create that would be translated into 133 bytes + headers. Base 64 is not secure and it adds unnecessary overhead. But would be ideal for a Http based application.
* Is slower than other native protocols as it needs to verify security and message formats of headers for each call.

In order to verify the above theory I tried to do performance tests and figure out the metrics

Code used to perf test using URLConnection

<script src="https://gist.github.com/vallur/1f062f73a3148565f6f9.js"></script>

Code for client thread to execute multiple gets at a time

<script src="https://gist.github.com/vallur/2a03147fd076ef0cbc94.js"></script>

Test results : -
```
total time spent for 2000000 calls - 24 mins 52 secs 177 ms 539 Us
```

Test results from JVM metrics :- 

![RestClient metrics](/img/posts/RestClient.png)

Metrics with the httpcomponents is way worse than using urlconnection so not showing that here. 

####WebSockets####

Websockets are a viable alternative to ReST API. But they are very simillar in the sense each request has similar kind of headers transfered back and forth. A socket can be upgraded and used for full duplex communication which is a biggest advantage over ReST. This advantage makes it viable to be used for writing a driver API for database based solutions. The biggest disadvantage with websockets is it is a very basic infrastructure on which we will have to build everything. If I have to choose WebSockets I can choose Sockets.

Test results : -
```
total time spent for 2000000 calls - 13 mins 45 secs 636 ms 583 Us
```

Test results from JVM metrics :- 

![RestClient metrics](/img/posts/WebSockets.png)

####Sockets####
I thought about my first java class where we built a chat application using plain old sockets.
When we want to support more than 50,000 requests per second per server, the biggest bottleneck is the size of the message sent back and forth over the network. A gigabit network can support a maximum number of bytes per second is ((1024^3)/8)*60% leaving 40% for other system monitoring overhead = 80,530,636 bytes. Any distributed No-SQL solution will have to deal with its own overhead if i assume we get 40% writes and 60% reads on a specific machine that means 40% of the requests have to have a hop to another machine to resolve conflict that leaves us with just 60% of the network of 80,530,636 = 48,318,381 bytes.

If we use 700 bytes for communication for each read/write we end up with 69,026 requests per second. I would say if the system keeps its data communication in this mark it should be ok in meeting the 50,000 to 70,000 requests per second per machine. If we assume that traces are run on the system we will be loosing a lot of bandwidth for that. It is safe to assume that by implementing using sockets and having very low overhead for point to point communication this system can support minimum 50,000 requests per second per node on a healthy state.

Picture below shows what the client server code does.
![Client server architecture for banyan](/img/posts/clientserver.png)

The code used to verify performance with client API calls.

<script src="https://gist.github.com/vallur/da13b54d6bd9b73567a2.js"></script>

Test results : - 
```
total time taken for completion of 2000000 calls - 8 mins 37 secs 780 ms 122 Us
```

Test results from JVM metrics :-

![SocketClient metrics](/img/posts/SocketClient.png)

Server side code for the socket 

<script src="https://gist.github.com/vallur/fd298d23db3730036b8c.js"></script>

The class responsible for servicing simple requests 

<script src="https://gist.github.com/vallur/133259b47e46a7240085.js"></script>

Test metrics from server side :-

![SocketClient metrics](/img/posts/socketServer.png)

From this, More than 50,000 requests per second is defenitely acheivable on the right hardware. 

Advantages:- 

* Behave very simmillar to ReST and can also listen to port 80.
* very light weight  
* Extreme performance on speed which allows to meet extreme amount of client requests per second.

Disadvantages:-

* will have to implement everything from scratch.
* Security will have to be implemented.
* Request and response is not human readable. Not exactly a disadvantage as Rest post requests are base 64 encoded which is not human readable either.

The hardware used for the performance tests are below:

 * Server - 2GHz Intel Core i7 16 GB 1600 DDR3 RAM - OSX 10.9.4
 * Client - 2.7 GHz Intel(R) Core i5 8 GB RAM - Windows 8.1
 * Java version - 1.8.0_05
 * Network Switch/Link speed - 100 Mbps. -- This is the major culprit for the slow speed.

The total round trip time depends on the network and i have seen average network lag of 300 Us to 9 ms per request for a communication bytesize of less than 1k for any of the above technologies we choose. This network lag begs for the need for compression, when the message size increases which i shall discuss in my next blog.

The above tests are not a apple to apple comparison as the tests on websockets and Rest API i used hello world example and in the socket case I used actual data gets from Banyan with byte size varying from 500 bytes to 1200 bytes of actual data including serialization and deserialization. 

For the reason of security and availability of admin operations it would not be a bad idea to implement a tomcat application on top of this framework. Just a thought for the future.

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
