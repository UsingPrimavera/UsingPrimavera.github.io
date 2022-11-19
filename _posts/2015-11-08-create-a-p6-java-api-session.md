---
layout: post
title:  Creating a P6 Java API Session
description: 'How to create a Primavera P6 Java API Session from which one can go on to write an application using the capabilities present in Primavera P6.' 
date:   2015-11-08 13:04:06 +0000
author:	barrie
categories: java-api
image: 'assets/images/p6_create-java-session400.png'
mermaid: true
---
First of all, before starting to write any code, one has to install the Oracle Primavera P6 Integration API. The official [Oracle Primavera P6 Documentation](https://docs.oracle.com/en/industries/construction-engineering/primavera-p6-eppm/index.html) will tell you how to do that. Having installed the P6 Integration API, the next step for any would be P6 programmer is to write code that will login to a P6 Java API Session so they can work their developer magic on P6.

No matter what else goes on in any application using the P6 Java API, they all have to create a session and log in, therefore, the class shown in this post should be useful. The P6 Java API can be run in local mode, where all the work is done on the same machine as the application, or it can be run in the remote mode where a web server does the heavy lifting and the application tells it what to do and gets the results back.

In this article, I'll share the class I use to get logged in to Primavera P6 in either mode and consequently get started fairly quickly in any environment. It's more of a helper class than anything else and as a result saves writing the same kind of code over and over again. The full code listing is at the end of this article, but first of all, I wanted to spend a little time just walking through it.

## All those Imports
The class is called __PrimaveraP6__ to distinguish it from other Primavera tools such as Unifier where I would call the equivalent class __PrimaveraUnifier__. The class definition starts off with all the imports that are needed to create a P6 Java API session. Anyone new to programming with the Oracle Primavera P6 Java API will soon learn there are a lot of imports required to do some simple things.

In the case of __PrimaveraP6__, the __Session__ and __RMIURL__ classes are used to login and there are three exception classes that can be thrown by the login method. In fact nearly every Primavera P6 class seems to throw __com.primavera.ServerException__, __com.primavera.integration.client.ClientExceptiPrimaveraon__ and __com.primavera.integration.network.NetworkException__.

There are two final imports as I make use of the Java logging classes. So here are all the imports used.

{% highlight java %}
import com.primavera.integration.client.Session;
import com.primavera.integration.client.RMIURL;
import com.primavera.ServerException;
import com.primavera.integration.client.ClientException;
import com.primavera.integration.network.NetworkException;

import java.util.logging.Logger;
import java.util.logging.Level;
{% endhighlight %}

The way the class is instantiated defines whether it will attempt to use the Remote or Local mode of the P6 Java API session. There are 3 constructors. The first has no parameters and is for the Local mode. The other two are for Remote mode, where one requires both the hostname and port, whilst the other requires the hostname and uses the default port of 9099. These constructors set some private members that are the most common configuration.

{% highlight java %}
* Manages the login to Primavera P6 over either remote or local mode. This
* class is a wrapper around the `Session` class. It adds some
* constructors to simplify handling connecting via local or remote mode.
*
* @author Barrie Callender
*/
public class PrimaveraP6 {

private final static Logger log = Logger.getLogger("com.usingp6.api.client.PrimaveraP6");
private String _url = null;
private int _rmiport = 9099;
private int _httpport = 0;
private int _mode = RMIURL.LOCAL_SERVICE;

/**
* Constructor for local mode only.
*
*/
public PrimaveraP6() {}

/**
* Constructor for remote where the hostname and port is supplied.
* The default mode is `STANDARD_RMI_SERVICE` and other modes will
* have to be set separately.
*
* @param url RMI registry host
* @param port RMI registry port number
*/
public PrimaveraP6(String url,int rmiport) {
_url=url;
_rmiport=rmiport;
_mode=RMIURL.STANDARD_RMI_SERVICE;
log.log(Level.CONFIG,"_url={0}, _rmiport={1},_mode={2}",new Object[]{_url,_rmiport,_mode});
}

/**
* Constructor for remote where only the hostname is supplied. The port
* defaults to 9099 and the mode is `STANDARD_RMI_SERVICE`.
* Other modes will have to be set separately.
*
* @param url RMI registry host
*/
public PrimaveraP6(String url) {
this(url,9099);
}
{% endhighlight %}

## Setting the right protocols
Once the `PrimaveraP6` class has been instantiated there are various methods to make changes like using SSL. There aren't a load of constructors to handle all these scenarios as most clients don't use SSL or HTTP tunnelling and I like simple classes that set the most common settings by default.

The reason most clients do not use SSL with Primavera P6 appears to be because there are so few people who know how to set it up. Most forget to configure the database to use SSL. Maybe a post for another day.

There is a method (`setSSL`) to set the Remote mode to use SSL and another method (`setStandard`) to not use SSL. There's even a method (`isSSL`) which reports if SSL is configured to be used or not.

{% highlight java %}
/**
* Set Remote Mode to use SSL.
*
*/
public void setSSL() {
_mode=RMIURL.SSL_RMI_SERVICE;
log.log(Level.CONFIG,"_mode={0}",_mode);
}

/**
* Set Remote mode to use standard protocol and not SSL.
*/
public void setStandard() {
_mode=RMIURL.STANDARD_RMI_SERVICE;
log.log(Level.CONFIG,"_mode={0}",_mode);
}
/**
* Returns true if SSL is being used in Remote Mode otherwise returns false;
* @return true if SSL is being used
*/
public boolean isSSL() {

boolean retval = false;
if (_mode==RMIURL.SSL_RMI_SERVICE) {
retval = true;
}
return retval;
}
{% endhighlight %}
In a similar fashion the `setHTTPPort` and `unsetHTTPPort` methods are used to set up or remove the configuration for HTTP tunnelling. (N.B. I have never used this and so it hasn't been tested).

## Login to Primavera P6
Finally the `login` method, which just calls `login` on the `Session` P6 Java API class. It returns an instance of the `Session` class which we can then use in our application. Typically I would use the class as in the following example, although the strings wouldn't be hard-coded:

{% highlight java %}
PrimaveraP6 localP6 = new PrimaveraP6();
localP6.login("admin","password","1");

PrimaveraP6 remoteP6 = new PrimaveraP6("myp6remoteserver"));
localP6.login("admin","password","1");
{% endhighlight %}

The class is pretty simple but can save a lot of time when trying to log in. It really deals with the more awkward aspects of setting up a connection. Using it means that it is very easy to create applications that can be configured at runtime to use either a Remote or a Local connection.

Here is a complete class and may your application development with the P6 Java API be a happy one.

{% highlight java %}
package com.usingp6.api.client;

import com.primavera.integration.client.Session;
import com.primavera.integration.client.RMIURL;
import com.primavera.ServerException;
import com.primavera.integration.client.ClientException;
import com.primavera.integration.network.NetworkException;

import java.util.logging.Logger;
import java.util.logging.Level;

/**
* Manages the login to Primavera P6 over either remote or local mode. This
* class is a wrapper around the `Session` class. It adds some
* constructors to simplify handling connecting via local or remote mode.
*
* @author Barrie Callender
*/
public class PrimaveraP6 {

private final static Logger log = Logger.getLogger("com.usingp6.api.client.PrimaveraP6");
private String _url = null;
private int _rmiport = 9099;
private int _httpport = 0;
private int _mode = RMIURL.LOCAL_SERVICE;

/**
* Constructor for local mode only.
*
*/
public PrimaveraP6() {}

/**
* Constructor for remote where the hostname and port is supplied.
* The default mode is `STANDARD_RMI_SERVICE` and other modes will
* have to be set separately.
*
* @param url RMI registry host
* @param port RMI registry port number
*/
public PrimaveraP6(String url,int rmiport) {
_url=url;
_rmiport=rmiport;
_mode=RMIURL.STANDARD_RMI_SERVICE;
log.log(Level.CONFIG,"_url={0}, _rmiport={1},_mode={2}",new Object[]{_url,_rmiport,_mode});
}

/**
* Constructor for remote where only the hostname is supplied. The port
* defaults to 9099 and the mode is `STANDARD_RMI_SERVICE`.
* Other modes will have to be set separately.
*
* @param url RMI registry host
*/
public PrimaveraP6(String url) {
this(url,9099);
}

/**
* Set Remote Mode to use SSL.
*
*/
public void setSSL() {
_mode=RMIURL.SSL_RMI_SERVICE;
log.log(Level.CONFIG,"_mode={0}",_mode);
}

/**
* Set Remote mode to use standard protocol and not SSL.
*/
public void setStandard() {
_mode=RMIURL.STANDARD_RMI_SERVICE;
log.log(Level.CONFIG,"_mode={0}",_mode);
}
/**
* Returns true if SSL is being used in Remote Mode otherwise returns false;
* @return true if SSL is being used
*/
public boolean isSSL() {

boolean retval = false;
if (_mode==RMIURL.SSL_RMI_SERVICE) {
retval = true;
}
return retval;
}

/**
* Sets the Remote Mode to use http tunnelling and supplies a port to use
*
* @param httpport - the HTTP port number which is used for HTTP/HTTPS tunneling mode only
*/
public void setHTTPPort( int httpport) {
_httpport=httpport;
log.log(Level.CONFIG,"_httpport={0}",_httpport);
}

/**
* Sets the Remode Mode to use http tunnelling with the default port set to 80
*
*/
public void setHTTPPort() {
setHTTPPort(80);
}

/**
* Switches off HTTP Tunnelling by setting the port to 0
*
*/
public void unsetHTTPPort() {
setHTTPPort(0);
}

/**
* Used to login to P6 by passing the UserName, Password and Database Instance
* Id. The constructor used to instantiate this class defines whether to
* connect via local or remote modes.
*
* Additional methods are provided to request the different remote modes and
* they should be called before this method.
*
* @param username - P6 User Name
* @param password - P6 Password
* @param databaseId - Database Id from list available
* @return Session session object
*/
public Session login(String username, String password, String databaseId) {

log.log(Level.CONFIG,"username={0},password={1},database id={2}",new String[]{username,password,databaseId});
log.log(Level.CONFIG,"_mode={0},_url={1},_rmiport={2},_httpport={3}",new Object[]{_mode, _url,_rmiport,_httpport});

Session retval=null;
try {
retval=Session.login(RMIURL.getRmiUrl(_mode, _url,
_rmiport,
_httpport),
databaseId,
username,
password);
}
catch(ServerException e) {
e.printStackTrace();
}
catch (ClientException e) {
e.printStackTrace();
}
catch (NetworkException e) {
e.printStackTrace();
}
return retval;
}

}
{% endhighlight %}

