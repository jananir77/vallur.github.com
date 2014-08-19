---
layout: post
title: Handling security in Java
category: Coding
tags: tomcat, security, java 8
year: 2014
month: 08
day: 8
published: true
summary: The first thing we need to worry about while developing a new software is handling security.

image: network-security1.png
---

The first thing we need to worry about while developing a new software is handling security.
There are many ways to solve this problem.

Active Directory
---
You can provide a unique encrypted client id for each client. The client and server will have to authenticate each other using the clientID and possibly using a secure channel using SSH or over SSL. In which case the server can accept the username and password and authenticate it towards LDAP using the code below

<script src="https://gist.github.com/vallur/0faa2623676185492e4b.js"></script>

If you are using a client API to communicate to server application. The client can access the logged on user name using the property to perform just authorization and not authentication by using the User Details from the LDAPAuthenticationProvider class above

```
System.getProperty("user.name")
```

Be mindful that the System.Property can be changed by the client java code so spoofing it from another java class is easy.

Single Sign On
---

It is not fair to close the topic without talking about Single sign on for security. It is difficult just as you might need to have the right permissions to create a key tab file for each server where your service is deployed. You can achieve this by using the <a href="http://docs.oracle.com/javase/jndi/tutorial/ldap/security/gssapi.html">GSS</a> and <a href="http://docs.oracle.com/javase/8/docs/technotes/guides/security/jaas/JAASRefGuide.html">JAAS</a> api.

We will be looking at Kerbros authentication with tomcat how-to.

Step 1 is the map a service principal name to a user account. A service principal name (SPN) is the name by which a client uniquely identifies an instance of a service.

```
SPN format: <service class>/<host>:<port>/<service name>
example:    Banyan/somehost.domain.com
```
In the SPN the port and service names are optional. Service SPN must be registered on the user or computer account that the service instance will use to log on. SPN registration is done by running with domain administrator privileges.

The setSPN command can be used to register a SPN for a service.

```
setspn -a banyan/somehost.domain.com domainUser
```
Now that we have registered the SPN we need to generate a key tab file that contains the private key for the service account which the server will use for authentication. Use the <a href="http://technet.microsoft.com/en-us/library/cc753771.aspx">ktpass</a> app to do this.

```
ktpass /out c:\tomcat.keytab /mapuser domainUser@SOMEDOMAIN.COM
          /princ banyan/somehost.somedomain.coml@SOMEDOMAIN.COM
          /pass password /kvno 0
```
Now we will have to create the krb5.ini and the JAAS.conf for authentication and authorization

KRB5.ini

```
[libdefaults]
default_realm = SOMEDOMAIN.COM
default_keytab_name = FILE: c:\tomcat.keytab 
default_tkt_enctypes = rc4-hmac,aes256-cts-hmac-sha1-96,aes128-cts-hmac-sha1-96
default_tgs_enctypes = rc4-hmac,aes256-cts-hmac-sha1-96,aes128-cts-hmac-sha1-96
forwardable=true

[realms]
SOMEDOMAIN.COM = {
        kdc = somehost.somedomain.com:88
}

[domain_realm]
somedomain.com= SOMEDOMAIN.COM
.somedomain.com= SOMEDOMAIN.COM
```
JAAS.conf file
```
com.sun.security.jgss.krb5.initiate {
    com.sun.security.auth.module.Krb5LoginModule required
    doNotPrompt=true
    principal="banyan/somehost.somedomain.coml@SOMEDOMAIN.COM"
    useKeyTab=true
    keyTab=" c:\tomcat.keytab"
    storeKey=true;
};

com.sun.security.jgss.krb5.accept {
    com.sun.security.auth.module.Krb5LoginModule required
    doNotPrompt=true
    principal="banyan/somehost.somedomain.coml@SOMEDOMAIN.COM"
    useKeyTab=true
    keyTab=" c:\tomcat.keytab"
    storeKey=true;
};
```

The krb5.conf location can be set by setting the property ```java.security.krb5.conf```. 
The JAAS.conf location can be set by setting the property ```java.security.auth.login.config```.

Now for the final part we need to change the server.xml and set the authentication valve.

Add the below setting to the server.xml file as part of the Host node 

```
<valve className="org.apache.catalina.authenticator.SpnegoAuthenticator" loginConfigName="com.sun.security.jgss.krb5.accept" />
```

By setting the above credentials tomcat will be able to authenticate using the browser using single sign on. Now we will be able to access the remote user from the servlet or asp using the code below.

```
request.getRemoteUser();
request.getUserPrincipal().getName();
```
This will work in both windows as well as linux.

Secure Shell
---
You can look @ <a href="http://en.wikipedia.org/wiki/Secure_Shell">wikipedea</a> about SSH and making secure connections between two servers. This is only a partial solution where you say who ever has access to the client/web server that communicates with your server has access to the data. The client still might need to do authentication and authorization. This is also a hammer as you will be incurring the cost of encryption and decryption on all the data communication between the servers.

Handle security in a custom way and this will not be single sign on and the data will be kind of secure  as long as no one gets hold of the client code and server code.

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
