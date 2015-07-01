Analytics: Lesson 1 - Load Generator
========================================


Overview
--------

Apigee Edge Analytics Services delivers the analytics tools and infrastructure that provides end-to-end visibility across the entire digital value chain. With Edge Analytics, enterprises can make data-driven decisions to grow the reach and revenue of your digital program, increase customer engagement, and accelerate digital transformation. In addition, Edge Analytics provides unmatched flexibility to meet changing business and analytics needs.

Edge Analytics does this by collecting a data record for each transaction or message that flows through Apigee Edge, and then computing aggregates on that data, and providing ready-to-use analytics charts and visualizations.  Edge Analytics also provides the ability to serve ad-hoc queries, and to build custom reports, to allow anyone to gain finely-tuned visibility into traffic trends and usage. 

Objectives
------------

In this lab you will get familiar with a tool that can be used to drive arbitrary contrived loads through APIs. As API Proxies defined in Apigee Edge handle the API traffic, Edge will collect the Analytics records, which allows you to visualize data charts on that data.  

We call this tool "the load generator" and is implemented as a nodejs program called runLoad.js that runs within Apigee Edge.  Running  runLoad.js as a Script Target within Apigee Edge means the load generator runs continuously, in the cloud.  The load generator is packaged within an API Proxy, and it acts as a client, for one or more APIs that you specify.  

Being able to generate a load on an API which varies over time in a somewhat random fashion is critical to being able to effectively demonstrate Apigee Edge Analytics. 

Prerequisites
--------------

- API Services - Lesson 1 completed
- The weather-quota API Proxy , which includes an oauth token verification policy.
- The oauth API Proxy , which dispenses oauth tokens via the client_credentials flow. 
- The runload-1 API Proxy , which contains the runLoad.js script which generates load
a tool to invoke single APIs, like curl, Postman, or Advanced REST Client

Estimated Time: 15 mins

1. Deploy the oauth proxy. Import the oauth proxy if necessary. After it is imported, verify the oauth proxy is deployed to the test environment. If it is not, deploy the oauth proxy to test. The oauth proxy will dispense tokens for other APIs in use in your organization.

  a. Open up a browser tab and log in to http://edge.apigee.com  
  b. From the Organization drop-down in the top-right corner, select the organization assigned to you  
  c. From the Environment drop-down, select "test"  
  d. From the main menu, select APIs   
  e. you should see a list of API Proxies. Click the "+ API Proxy" button
  ![Plus API Proxy](https://raw.githubusercontent.com/DinoChiesa/se-partner-bootcamp/master/loadgen/images/plus-API-Proxy.png)

  f. In the New API Proxy panel, select "API Bundle". 
  ![New API Proxy](https://raw.githubusercontent.com/DinoChiesa/se-partner-bootcamp/master/loadgen/images/New-API-Proxy.png)

  g. Select "Choose file" and upload the oauth.zip bundle . The name of the proxy will be pre-populated with the word "oauth".  That's good.  Click "Build".

  h. click "Close" when the build completes. 

  i. In the resulting list of API Proxies, click the "oauth" proxy. 

  j. Click the Deployment drop-down, and select "test".  This will deploy the oauth proxy to the test environment.
  ![New API Proxy](https://raw.githubusercontent.com/DinoChiesa/se-partner-bootcamp/master/loadgen/images/deployment-dropdown.png)

2. Import and Deploy the weather-quota apiproxy the same way.  

3. Import the runload-1 apiproxy the same way. EXCEPT - do not deploy the runload-1 proxy.  We'll deploy it in a few moments. 

4. Using the "new API Proxy editor", Examine the oauth proxy you have imported - you will see that it has exactly one resource: the AccessTokenClientCredential, which accepts a POST to /token. You will use this to request a bearer token which will be verified by the weather-quota proxy. 
![oauth proxy](https://raw.githubusercontent.com/DinoChiesa/se-partner-bootcamp/master/loadgen/images/oauth-proxy.png)

5. ok, now examine the weather-quota proxy in the same way.  You will see that it has two flows - the first  queries the weather. This request is protected by an oauth token verification.  The second is a "default flow" which matches all other inbound requests.  This one just returns an "Unknown request" error (404)  to the caller. 

![weather-quota proxy](https://raw.githubusercontent.com/DinoChiesa/se-partner-bootcamp/master/loadgen/images/weather-quota-proxy.png)

6. Now examine the runload-1 proxy, in the API Proxy editor (the new version).  Scroll the left-hand-side project navigator panel all the way down.  Select the runLoad.js file. This is the file that contains the generic logic for the load generator. You can browse the source of this nodejs module. It's about 36k of code. 

![runload-1 proxy](https://raw.githubusercontent.com/DinoChiesa/se-partner-bootcamp/master/loadgen/images/runload-1-proxy-runloadjs.png)

7. You don't need to understand all the code, but you should be aware of the design of runload.js, which is as follows:

  a. The runload nodejs server starts up and "listens" on a control interface.  
  b. In the callback to the listen() function, it sets a timeout.   
  c. in the function called by the setTimeout(), runload reads a configuration file, then sends out a batch of API calls as described in that configuration file.  
  d. The function then sets another timeout, for some delay. About 60 seconds by default, but this can vary depending on the configuration.   
  e. the timeout fires, and runload again sends out a batch of API calls.   
  f. this continues indefinitely, or until you administratively "undeploy" the proxy, or send a "stop" command to the runload server. 
  
8. Now, back in the Edge UI, Select the model.json file in that lower panel.  

![runload-1 proxy](https://raw.githubusercontent.com/DinoChiesa/se-partner-bootcamp/master/loadgen/images/runload-1-proxy-modeljson.png)

9. This shows some basic configuration for runload. The code looks like this: 

```
{
  "id" : "job1",
  "description": "drive some test APIs in the sandbox organization",
  "loglevel" : 3,
  "defaultProperties": {
    "scheme": "https",
    "host": "NAME-sandbox-test.apigee.net",
    "headers" : {
      "Accept" : "application/json"
    }
  },

  "initialContext" : {
    "client_id" : "CLIENT_ID_HERE",
    "client_secret": "CLIENT_SECRET_HERE"
  },

  "sequences" : [{
    "description" : "create-token",
    "iterations" : 1,
    "requests" : [ {
      "method" : "post",
      "url" : "/oauth/token",
      "payload" : "grant_type=client_credentials&client_id={client_id}&client_secret={client_secret}"
    }]
  }]
}

```





