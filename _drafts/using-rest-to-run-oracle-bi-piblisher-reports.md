---
layout: post
title: Using REST to Run Oracle BI Piblisher Reports
date: 2022-02-20T20:32:40+00:00
description: This post explains how to retrieve data using REST to run Oracle BI Publisher reports.  It begins where the "REST API for Oracle Business Intelligence Publisher" left off.
author: barrie
image: assets/images/postman_running_bip_report.png
---
Recently we needed to run a series of Oracle BI Publisher reports to extract the data and store it in another system. We found the official [REST API for Oracle Business Intelligence Publisher](https://docs.oracle.com/middleware/bi12214/bip/BIPAP/index.html) to be incomplete in this respect.  We managed to hunt down the missing information, and decided to write this post to help others who may be stalled in the same way we were.

We used the free [Postman](https://www.postman.com/) application to find out what to do, and have also used that knowledge to implement the same thing using [PowerBI Desktop](https://www.microsoft.com/en-gb/p/power-bi-desktop/9ntxr16hnw1t?activetab=pivot:overviewtab) and the [Power Query M formula language](https://docs.microsoft.com/en-us/powerquery-m/).

This post will start by using the REST API directly in a Web Browser to retrieve information about a report.  Then we'll do the same thing using Postman to ease you into it.  Then we'll use Postman to run a report to return some data, before runninning a report that needs a parameter.  We'll finish up with how to get data in different formats.  By this article we hope you will have the confidence to make use of the BI Publisher REST service.

This article will not show how we extracted the data sent to us from BI Publisher and place it somewhere else.  We'd like to save that for future posts.

## The state of the documentation for the REST API for BI Publisher
![BI Publisher Documentation Banner](/assets/images/oracle_bip_docs_banner.png){:class="img-responsive"}
REST is an acronym that stands for **RE**presentational **S**tate **T**ransfer and is an architectural style that was first presented in 2000.  We're not going to go into the depths of what exactly that means, but here are a few resources we have found useful, but please carry on with this article and come back to these and other resources later if you need or want to increase your knowledge:

* [What is REST](https://www.codecademy.com/article/what-is-rest)
* [REST API Tutorial](https://restfulapi.net/)
* [Representational state transfer - Wikipedia](https://en.wikipedia.org/wiki/Representational_state_transfer)

We'll try and introduce everything you need to know as we go along.  The documentation for the REST API for BI Publication has some essential information, but assumes you know REST and the [HTTP Protocol](https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol), so it is a good start.  Unfortunately, it lets itself down by some poor examples.  It does tell us where options can be provided, but doesn't mention what those options are or how to format them.

The state of the documentation is a case of close, but not complete.

## Get a report definition
![Get Report Definition](/assets/images/get_report_definition_bip_docs.png)
Retrieving the report definition is the easiest thing to do, and can be done using just a web browser.  I entered ```http://ple-2012-bi:9502/xmlpserver/services/rest/v1/reports/~barrie%2FBIPREST%2FPARAM_REPORT``` into my web browser.  It then asked me for my credentials for BI Publisher and returned the following in the web browser.

{% highlight json %}
{"ESSJobName":"","ESSPackageName":"","autoRun":true,"cacheDocument":true,"controledByExtApp":false,"dataModelURL":"/~weblogic/BIPREST/DATA_MODELS/PARAM_REPORT.xdm","defaultOutputFormat":"csv","defaultTemplateId":"PARAM_REPORT","diagnostics":false,"listOfTemplateFormatsLabelValues":[{"active":true,"applyStyleTemplate":true,"default":true,"listOfTemplateFormatLabelValue":[{"templateFormatLabel":"Data","templateFormatValue":"xml"},{"templateFormatLabel":"CSV","templateFormatValue":"csv"}],"templateAvailableLocales":["en_US"],"templateDefaultLocale":"en_US","templateID":"PARAM_REPORT","templateType":"xpt","templateURL":"PARAM_REPORT.xpt","viewOnline":true}],"onLine":true,"openLinkInNewWindow":true,"parameterColumns":3,"parameterNames":["p6ScheduleObjectId"],"reportDefnTitle":"","reportName":"PARAM_REPORT","reportParameterNameValues":[{"UIType":"Text","dataType":"xsd:integer","defaultValue":"38","fieldSize":"15","label":"P6 Internal Project Id","multiValuesAllowed":false,"name":"p6ScheduleObjectId","refreshParamOnChange":false,"selectAll":false,"templateParam":false,"useNullForAll":false}],"showControls":true,"showReportLinks":true,"templateIds":["PARAM_REPORT"]}
{% endhighlight %}

It might not look very exciting, but it is a nice easy start and you can do it just to see if you can reach your BI Publisher.  

Let's break that URL down.

### What's in a URL?
The first part of the URL is ```http://ple-2012-bi:9502/xmlpserver``` which would take us to the BI Publisher application if that was all we entered.  The next part is documented by Oracle in the [Get Report Definition](https://docs.oracle.com/middleware/bi12214/bip/BIPAP/op-v1-reports-reportpath-get.html)) as ```/services/rest/v1/reports/{reportPath}```.  This is the REST specific part of the whole URL.  The REST implementation on BI Publisher makes use of URLs in combination with parameters, payloads and different method types to carry out different requests.  

The Oracle documentation for the Get Report Definition request says it uses the ```Get``` method, which is the easiest to run by typing it in a browser, just like many other requests we make on a regular basis using our web browsers.  5 of the 14 endpoints in the BI Publisher REST service are Get requests.  It is almost a case of just typing each of them into your web browser to get something back.

The last part of the endpoint is the ```{reportPath}``` parameter.  As it's name suggests it is the path to the report, whose definition we are after, but it doesn't look like it at first sight.  The path, as seen in BI Publsiher is ```~barrie/BIPREST/PARAM_REPORT```.  The Report is called ```PARAM_REPORT``` and is in the ```BIPREST``` folder inside the "My Folders" directory which is my username ```~barrie```.  Parameters can't contain certain characters that are used in a normal URL.  They have to be encoded by being replaced with something else.  A forward slash can't be used so it is replaced by ```%2F```, and a space can't be used so it is replaced by ```%20```.  There are others, but luckily most programming languages have a library that can URL-encode your parameters for you.

The documentation places a parameter name between curly braces, and it has to be replaced with a URL encoded version of the value of the parameter.  The 5 Get requests require the ```{reportPath}```, ```{templateId}``` parameter or both.  In the report we use here, the {templateId} is the same name as the report.

### Get these with just your web browser
I can enter any of the five Get requests into my browser to get something back. 

* Get Report Definition: /services/rest/v1/reports/{reportPath} as ```http://ple-2012-bi:9502/xmlpserver/services/rest/v1/reports/~barrie%2FBIPREST%2FPARAM_REPORT```
* Get Report Sample Data: /services/rest/v1/reports/{reportPath}/sampleData which downloads a file if the report has sample data as ```http://ple-2012-bi:9502/xmlpserver/services/rest/v1/reports/~barrie%2FBIPREST%2FPARAM_REPORT/sampleData```
* Get Report Template: /services/rest/v1/reports/{reportPath}/templates/{templateId} which downloads a file as ```http://ple-2012-bi:9502/xmlpserver/services/rest/v1/reports/~barrie%2FBIPREST%2FPARAM_REPORT/templates/PARAM_REPORT```
* Get Report Template Parameters: /services/rest/v1/reports/{reportPath}/templates/{templateId}/parameters as ```http://ple-2012-bi:9502/xmlpserver/services/rest/v1/reports/~barrie%2FBIPREST%2FPARAM_REPORT/templates/PARAM_REPORT/parameters```
* Get XDO Schema: /services/rest/v1/reports/{reportPath}/xdoSchema which downloads a file as ```http://ple-2012-bi:9502/xmlpserver/services/rest/v1/reports/~barrie%2FBIPREST%2FPARAM_REPORT/xdoSchema```

BI Publisher will ask for your credentials the first time, and so long as you don't leave it to long, it will let you keep sending Get Requests.

The truth is we don't need to use any of these Get requests to just run a report, but we wanted to show the basics and then build up from there.  

## A little help from Postman
In this part of the post we are going to use the free [Postman](https://www.postman.com/) application to implement the Get Report Definition, and continue using it for the rest of the post.  It is an application that can be used to build and test web services and get to see what is going on.  We're going to use a small part of it's capabilities to demonstrate how to use the BI Publisher REST API.  [SoapUI](https://www.soapui.org/) is another app that with similar intentions.

The first thing we did was [create an Environment](https://learning.postman.com/docs/sending-requests/managing-environments/) in Postman and set some variables and their initial values to use throughout our work. We setup the following four variables:

* ```base_url``` = ```ple-2012-bi``` - the first part of our URL
* ```rest_service``` = ```/xmlpserver/services/rest/v1/reports``` - the next part of the URL that is common to all the REST API URLs
* ```username``` = ```your BIP username``` - our username for BI Publisher
* ```password``` = ```your BIP password``` - the password for our username. We set the type of this one to "secret" so nobody can see what it is.

![Setting environment variables](/assets/images/postman_setting_environment_variables.png)

Having created our variables we selected our environment from the list in the top right-hand side and all those variables are now available for us to use.

We then [created a Collection](https://learning.postman.com/docs/getting-started/creating-the-first-collection/) in Postman which is a place to store the requests we will be making.  The Collection uses Basic Authorization.  We selected **Basic Auth** from the **Type** drop-down and then entered **{{username}}** and **{{password}}** in the credentials so Postman can pick them up from our Environment.  Whenever we want to use the contents of a variable, we can enter the name of it surrounded by double curly brackets.
![Create Postman collection using Basic Authorization](/assets/images/postman_create_collection_authorisation.png)
In addition to setting the Authorization Type, we setup the URL Encoded version of our path.  This is very simple to do with Postman.  We enter the path as we would normally see it, highlight what we have entered and select **Encode URI Component** from the drop-down.  There is a **Decode URI Component** to swap it back if we need to.

We're now ready to create a Get Report Definition request in Postman.

