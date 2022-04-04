---
layout: post
title: "Run Oracle BI Publisher reports using REST"
date: 2022-04-04T20:44:32+00:00
description: This first post takes the reader to the point of making REST GET requests to BI Publisher using Postman.  The next post in the series shows how to run reports.
author: barrie
categories:
image: assets/images/postman_running_bip_report.png
---
Recently we needed to run a series of Oracle BI Publisher reports to extract the data and store it in another system. We found the official [REST API for Oracle Business Intelligence Publisher](https://docs.oracle.com/middleware/bi12214/bip/BIPAP/index.html) to be incomplete in this respect.  We managed to hunt down the missing information, and decided to write a post showing what we did in an effort to help others who may be stuck in the same way we were.

As the original post grew, we decided it would be better to write more than one post, and publish them at the same time.

This is the first post in the series and starts by using the REST API directly from a Web Browser to retrieve information about a report.  Then we'll do the same thing using Postman to ease you into how it works.  The next post in this short series uses Postman to run a report to return some data, before running reports that need parameters and how to retrieve the same report in different data formats.

We used the free [Postman](https://www.postman.com/) application to find out what to do, and have also used that knowledge to implement the same thing using [PowerBI Desktop](https://www.microsoft.com/en-gb/p/power-bi-desktop/9ntxr16hnw1t?activetab=pivot:overviewtab) and the [Power Query M formula language](https://docs.microsoft.com/en-us/powerquery-m/).

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
Retrieving the report definition is the easiest thing to do, and can be done using just a web browser.  We entered ```http://ple-2012-bi:9502/xmlpserver/services/rest/v1/reports/~barrie%2FBIPREST%2FPARAM_REPORT``` into a web browser.  It then asked us for some credentials for BI Publisher and returned the following in the web browser.

{% highlight json %}
{"ESSJobName":"","ESSPackageName":"","autoRun":true,"cacheDocument":true,"controledByExtApp":false,"dataModelURL":"/~weblogic/BIPREST/DATA_MODELS/PARAM_REPORT.xdm","defaultOutputFormat":"csv","defaultTemplateId":"PARAM_REPORT","diagnostics":false,"listOfTemplateFormatsLabelValues":[{"active":true,"applyStyleTemplate":true,"default":true,"listOfTemplateFormatLabelValue":[{"templateFormatLabel":"Data","templateFormatValue":"xml"},{"templateFormatLabel":"CSV","templateFormatValue":"csv"}],"templateAvailableLocales":["en_US"],"templateDefaultLocale":"en_US","templateID":"PARAM_REPORT","templateType":"xpt","templateURL":"PARAM_REPORT.xpt","viewOnline":true}],"onLine":true,"openLinkInNewWindow":true,"parameterColumns":3,"parameterNames":["p6ScheduleObjectId"],"reportDefnTitle":"","reportName":"PARAM_REPORT","reportParameterNameValues":[{"UIType":"Text","dataType":"xsd:integer","defaultValue":"38","fieldSize":"15","label":"P6 Internal Project Id","multiValuesAllowed":false,"name":"p6ScheduleObjectId","refreshParamOnChange":false,"selectAll":false,"templateParam":false,"useNullForAll":false}],"showControls":true,"showReportLinks":true,"templateIds":["PARAM_REPORT"]}
{% endhighlight %}

It might not look very exciting, but it is a nice easy start, and doing it confirms you can use REST to reach your BI Publisher system.  

Let's break that URL down.

### What's in a URL?
The first part of the URL is ```http://ple-2012-bi:9502/xmlpserver``` which would take us to the BI Publisher application if that was all we entered.  The next part is documented by Oracle in the [Get Report Definition](https://docs.oracle.com/middleware/bi12214/bip/BIPAP/op-v1-reports-reportpath-get.html) as ```/services/rest/v1/reports/{reportPath}```.  This is the REST specific part of the whole URL.  The REST implementation on BI Publisher makes use of URLs in combination with parameters, payloads and different method types to carry out different requests.  They all follow the hostname and port fragment with ```/xmlpserver/services/rest/v1/reports/```.  Anything after this, such as ```{reportPath}``` is specific to the request being made.

The Oracle documentation for the Get Report Definition request says it uses the **Get** method, which is the easiest to run by typing it in a browser, just like many other requests we make on a regular basis using our web browsers.  5 of the 14 endpoints in the BI Publisher REST service are Get requests.  It is almost a case of just typing each of them into your web browser to get something back.

The last part of this Get Report Definition endpoint is the **{reportPath}** parameter.  As it's name suggests it is the path to the report, whose definition we are after, but it doesn't look like it at first sight.  The path, as seen in BI Publsiher is ```~barrie/BIPREST/PARAM_REPORT```.  The Report is called **PARAM_REPORT** and is in the **BIPREST** folder inside the "My Folders" directory which is my username **~barrie**.

Parameters can't contain certain characters that are used in a normal URL.  They have to be encoded by being replaced with something else.  A forward slash can't be used so it is replaced by **%2F**, and a space can't be used so it is replaced by **%20**.  There are others, but luckily most programming languages have a library that can URL-encode parameters.

The documentation places a parameter name between curly braces, and it has to be replaced with a URL encoded version of the value of the parameter.  The 5 Get requests require the **{reportPath}**, **{templateId}** parameter or both.  In the report we use here, the **{templateId}** is the same name as the report.

### Get these with just your web browser
I can enter any of the five Get requests into my browser to get something back. 

* Get Report Definition: /services/rest/v1/reports/{reportPath} as ```http://ple-2012-bi:9502/xmlpserver/services/rest/v1/reports/~barrie%2FBIPREST%2FPARAM_REPORT```
* Get Report Sample Data: /services/rest/v1/reports/{reportPath}/sampleData which downloads a file if the report has sample data as ```http://ple-2012-bi:9502/xmlpserver/services/rest/v1/reports/~barrie%2FBIPREST%2FPARAM_REPORT/sampleData```
* Get Report Template: /services/rest/v1/reports/{reportPath}/templates/{templateId} which downloads a file as ```http://ple-2012-bi:9502/xmlpserver/services/rest/v1/reports/~barrie%2FBIPREST%2FPARAM_REPORT/templates/PARAM_REPORT```
* Get Report Template Parameters: /services/rest/v1/reports/{reportPath}/templates/{templateId}/parameters as ```http://ple-2012-bi:9502/xmlpserver/services/rest/v1/reports/~barrie%2FBIPREST%2FPARAM_REPORT/templates/PARAM_REPORT/parameters```
* Get XDO Schema: /services/rest/v1/reports/{reportPath}/xdoSchema which downloads a file as ```http://ple-2012-bi:9502/xmlpserver/services/rest/v1/reports/~barrie%2FBIPREST%2FPARAM_REPORT/xdoSchema```

BI Publisher will ask for your credentials the first time, and so long as you don't leave it too long, it will let you keep sending Get Requests.

The truth is we don't need to use any of these Get requests to just run a report, but we wanted to show the basics and then build up from there.  

## A little help from Postman
In this part of the post we are going to use the free [Postman](https://www.postman.com/) application to implement the Get Report Definition, and continue using it for the rest of the post.  It is an application that can be used to build and test web services and get to see what is going on.  We're going to use a small part of it's capabilities to demonstrate how to use the BI Publisher REST API.  [SoapUI](https://www.soapui.org/) is another app with similar capabilities.  We use Postman because we found it easier to find out how to use it than we did with SoapUI.

The first thing we did was [create a Collection](https://learning.postman.com/docs/getting-started/creating-the-first-collection/) in Postman which is a place to store the requests we will be making.  We gave the Collection a name and then setup Authorisation.  The Collection uses Basic Authorization.  Under the **Authorization** Tab, we selected **Basic Auth** from the **Type** drop-down and then entered our credentials for BI Publisher in the **Username** and **Password** text boxes.

![Create Postman collection using Basic Authorization](/assets/images/postman_create_collection_authorisation.png)
In addition to setting the Authorization Type, we setup the URL Encoded version of our path.  This is very simple to do with Postman.  We enter the path as we would normally see it, highlight what we have entered and select **Encode URI Component** from the drop-down.  There is a **Decode URI Component** to swap it back if we need to.

We're now ready to create a Get Report Definition request in Postman.

## The GET Request for the Report Definition

[Sending a Request](https://learning.postman.com/docs/getting-started/sending-the-first-request/) is very easy with Postman.  Open a new Tab by clicking on the + button near the top of Postman.  The new tab defaults to a **GET** request and it is possible to just type the full URL directly into the text box.  The **{reportPath}** is a URL Encoded path, which is very easy for us to do as our path is ```~barrie/BIPREST/PARAM_REPORT```. Not all paths are this easy to convert, but Postman provides a helping hand.  We entered the full URL as ```http://ple-2012-bi:9502/xmlpserver/services/rest/v1/reports/~barrie/BIPREST/PARAM_REPORT```.  We then highlighted the part representing the path to the report, clicked the elipses next to the **Set as variable** that pops up and selected **Encode URI Component** from the drop-down.  There is also a **Decode URI Component** to swap it back if needed.

Our URL now looks like this: ```http://ple-2012-bi:9502/xmlpserver/services/rest/v1/reports/~barrie%2FBIPREST%2FPARAM_REPORT```.

The Request has to be saved to our Collection, by selecting **Save As** from the Save button drop-down.  Once this is done we moved to the ```Authorization``` Tab of the Request and changed the Type to **Basic Auth**.

We then clicked the **Send** button and the definition was returned to Postman.

We are using hardly any of the functionality of Postman here, clicking around the various tabs will reveal a lot of information about the request being made and the responses returned.

## Running a simple report
![BI Publisher REST Documentation banner for Run Report](/assets/images/bip_run_report_rest_endpoint.png){:class="img-responsive"}
By a simple report we mean one that does not require any parameters.  Running reports makes use of the **POST** method and we also have to send some information using a Multipart Form.

The BI Publisher REST documentation to [run a report](https://docs.oracle.com/middleware/bi12214/bip/BIPAP/op-v1-reports-reportpath-run-post.html) specifies the format of the url as ```/services/rest/v1/reports/{reportPath}/run```, which just like the Get Report Definition request is what appears after the normal URL for BI Publisher.  The URL request is the same as for the Get Report Definition except it ends with ```/run```.

We went to Postman and opened a new tab by clicking on the + button.  We entered the URL by copying the Get report Definition URL into the text box and adding ```/run``` to the end of it.  The Run Report request has to be a **POST** method so we changed it from **GET** to **POST**.
The **Content-Type** has to be **multipart/form-data**. To set that up we went to the **Headers** tab and entered **Content-Type** as the **Key** and **multipart/form-data** as the **Value**.  Both of these appeared as values to choose once we started typing.  
![Setting the Headers in Postman](/assets/images/postman_bip_run_headers.png)

The Run Report request needs to send a ```ReportRequest``` as a part in the request which requires it being added to the body of the request.  This is sometimes called the Payload.  There are many attributes that can go in here but we are going to keep it very simple at first, and discuss some of them later.

We switch to the **body** tab where we need to click on the **form-data** radio button before we can enter the ReportRequest which requires us to add the **Key**, **Value** and **Content-Type**.  Postman doesn't display the last of these by default so we have to include Content-Type by clicking the elipses on the headings row and ensuring **Content Type** is selected, then we can enter the required values.  In our case ```ReportRequest``` is typed into the **Key** box, ```{"byPassCache":true}``` in the **Value** box and ```application/json``` into the **Content Type** box.  The Value is in Json format so we have to specify the content as application/json.
![Setting the values in the body for the multipart form](/assets/images/postman_bip_simple_reportrequest.png)

Just as with the Report Definition, the last steps are to save the Request to our collection and set the Authorization, before clicking on the **Send** button to get the report data.  The screenshot shows the data returned as CSV data.
![Contents of data returned as CSV](/assets/images/postman_bip_simple_csv_data.png)

Our simple report defaults to returning data as CSV, but it is also configured to return data as XML.  We can request the report is returned as XML by passing ```"attributeFormat":"xml"``` as part of the ```ReportRequest``` so now the value of the ReportRequest is ```{"byPassCache":true, "attributeFormat":"xml"}```.  This time the data is returned in xml format when the Send button is clicked.
![Contents of data returned as XML](/assets/images/postman_bip_simple_xml_data.png)

There are many options that can be passed, but they aren't documented in the REST documentation.  They can be found in the SOAP Web Services documentation which is in the [Developer's Guide for Oracle Business Intelligence Publisher](https://docs.oracle.com/middleware/12213/bip/BIPDV/index.html).  We found them in the [2.3.46 ReportRequest](https://docs.oracle.com/middleware/12213/bip/BIPDV/datatypes.htm#BIPDV209) section.  If the report has not been configured to use an option found here, then requesting it won't make a difference.  As an example, our report is not configured to return a PDF format so setting the attributeFormat to pdf results in an error being returned.

## Requesting a report with a parameter
The report we have been using just returns some basic information about Activities in the database.  We have now changed the report to require a parameter for the internal P6 Project Id.  The report will now return only the P6 Activities for that specific P6 Project.

The parameter is named ```p_proj_id``` with a **Data Type** of **Integer** and a **Parameter Type** of **Text**.  This just means we can send it an integer value and so long as there is a P6 project with a proj_id of the value we send, then a record for each task will be returned.  If there is no project then BI Publisher will return an empty XML stream, that will look like this: ```<data_ds></data_ds>```.

The parameter is passed as another value in the ```ReportRequest```.  We need to send the value **411** to the **p_proj_id** parameter, so we need to add ```"parameterNameValues": { "listOfParamNameValues": [{"name": "p_proj_id","values": "411"}]}``` into the ReportRequest.

It is very difficult to work this out from the BI Publisher REST documentation, which seems to send us round and round in circles.  It is slightly less difficult to get it from the BI Publisher SOAP documentation.  Considering it is something that is very common, we think it needs adding as an example to the Run Report API.

The ```name``` is the name of the BI Publisher, which in our case is ```p_proj_id```, and the ```values``` is the value we wish to set.  To test this out we added the parameterNameValues structure to the end of the existing **ReportRequest** string, separating it with a comma.  The full string now looks like this: ```{"byPassCache":true, "attributeFormat":"xml", "parameterNameValues": { "listOfParamNameValues": [{"name": "p_proj_id","values": "411"}]}}```

When we click the **Send** button in Postman it now returns the tasks for the P6 Project with an internal P6 Project Id of 411.

If we want to get the tasks for more than one P6 project, we can call the report for each project, but that seems a little cumbersome.  BI Publisher allows the parameter value to be a set of comma separated values.  We made the change to our report by selecting the option ```Text field contains comma-separated values``` and then updated the WHERE statement from ```WHERE proj_id = :p_proj_id``` to be ```WHERE proj_id IN ():p_proj_id)```.

We clicked the **Send** button in Postman to verify it still returns the tasks for a single project, which it did.  Now we decided to request the tasks for 3 different projects with project_id of 412, 413 & 414.  All we did was replace the parameter value of **411** with **412,413,414** so the full **ReportRequest** is now set to: ```{"byPassCache":true, "attributeFormat":"xml", "parameterNameValues": { "listOfParamNameValues": [{"name": "p_proj_id","values": "412,413,414"}]}}```.

We clicked the **Send** button and it returned the tasks from the project specified.

## What next?
Anyone who has been struggling with working out how to run a BI Publisher report using the REST API, and then pass a parameter to retrieve data should be able to do that using the information in this post.  There is a lot more to try out and explore, such as passing more than one parameter, or retieving a list of values from BI Publisher and setting one or more of them them as parameters in the **ReportRequest**.

We plan to visit those topics in another post, and also show how we extracted the XML and put it in a database.
