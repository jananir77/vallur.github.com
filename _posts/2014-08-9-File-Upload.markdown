---
layout: post
title: File upload using java servlet and Tomcat
category: Coding
tags: java, tomcat, JVM
year: 2014
month: 08
day: 8
published: true
summary:  As part of this blog we will look at how to upload and process a file using java Servlet API. I will not be using the Apache File-Upload API.

image: apache.png
---
As part of this blog we will look at how to upload and process a file using java Servlet API.
I will not be using the Apache File-Upload API.

When  defining your servlet determine a folder where you want the temporary file to be downloaded.

you can have the below MultipartConfig annotation as part of the Servlet class file like below.

```
@MultipartConfig(location="/tmp", fileSizeThreshold=-1L, maxFileSize=10*1024*1024, maxRequestSize=10*1024*1024)
public class FileUploaderServlet extends HttpServlet
{
```
or you can have it as a configuration as part of the web.xml file. 

```
<multipart-config>
      <location>/tmp</location>
      <max-file-size>10485760</max-file-size>
      <max-request-size>10489760</max-request-size>
      <file-size-threshold>-1L</file-size-threshold>
</multipart-config>
```

If we are developing a general purpose file up-loader using the configuration in web.xml is the recommended approach. There are default values for all of the above which unlimited for all parameters . In my example my threshold is greater than the max-file-size so everything is instantaneous and in memory.

We are giving a sledge hammer here and the user can provide a large enough file to block the network. I would prefer the file size to be a max of 10 MB which is close to 10% of a Gigabit network traffic for a second. Any amount more than that will have the potential to create issues when more users are trying to download or upload contents to a server which is already processing other requests and potentially bring the server to a crawl for a few seconds.

Now for the code part 

<script src="https://gist.github.com/vallur/1ce92f3df5347ae2907a.js"></script>

The below are the steps performed

* Verify if the request is multipart if not just return.
* Prepare up the metadata information to parse the csv file.
* Use the FilePart from Part to get the inputstream that holds the stream information.
* Always use a BufferedReader on the InputStream as that would further mitigate the load on the network based on the time taken for processing.
* We read through the file untill we get a line that is null(End of File). It is possible to get a null in the beginning so we read the headers inside of the loop.
* Merge the results coming from the file into Banyan.

I had introduced a new set of items IBranch and DataBranch these are the basic building blocks of banyan which is built on top of the ArrayMap interface. Banyan uses a modified version of the above class to upload contents of a csv into the data store. Banyan also supports other formats like JSON but not through this example.

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
