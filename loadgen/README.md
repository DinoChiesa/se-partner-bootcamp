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

Estimated Time: 40 mins

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

4. Using the "new API Proxy editor", examine the oauth proxy you have imported - you will see that it has exactly two flows: the first is named AccessTokenClientCredential, which accepts a POST to /token. Apps can use this to request a bearer token which will be verified by the weather-quota proxy. The second is a "default flow" which matches all other inbound requests.  This one just returns an "Unknown request" error (404)  to the caller. 
![oauth proxy](https://raw.githubusercontent.com/DinoChiesa/se-partner-bootcamp/master/loadgen/images/oauth-proxy.png)

5. Now examine the weather-quota proxy in the same way.  You will see that it has two flows - the first queries the weather. This request is protected by an oauth token verification.  The second is again an "unknown request" flow, which gets executed for all other requests. 
![weather-quota proxy](https://raw.githubusercontent.com/DinoChiesa/se-partner-bootcamp/master/loadgen/images/weather-quota-proxy.png)

6. In the Edge UI, create a new Product, containing the weather-quota proxy: 

  a. Click "Publish", and select "Products" from the dropdown  
  b. click "+Product"  
  c. name the product "WeatherQuota-1"  
  d. tick the "test" checkbox  
  e. mark it public   
  f. set the quota to be 1000 requests for every 1 minute  
  g. ignore the oauth scopes section; it is not used in this exercise.  
  h. add a proxy by clicking the "+ API Proxy" button
  ![new product](https://raw.githubusercontent.com/DinoChiesa/se-partner-bootcamp/master/loadgen/images/create-new-product-20150630-211843.png)  

  i. select the weather-quota apiproxy.  
  j. click Save to save the API Product

7. Create a new Developer App that has access to this new product. 

  a. Click "Publish", then select "Developer Apps" from the dropdown.  
  b. Click "+ Developer App"  
  c. name the app "wq-app-1"  
  d. select any developer you like from the dropdown  
  e. ignore the callback URL; it is not used for this exercise.   
  f. click "+ Product"
  ![new app](https://raw.githubusercontent.com/DinoChiesa/se-partner-bootcamp/master/loadgen/images/create-developer-app-add-product-20150630-212308.png)  
  g. select the WeatherQuota-1 product you just created.  
  h. be sure to click the checkmark on the right-hand-side of the form  
  i. click Save  

8. Get the client_id and client_secret for that app.  

  a. In the resulting list, click the top item - it should be the wq-app-1 you just created.  
  b. show the client_id and client_secret.  Copy these values to a text file.  



9. Now examine the runload-1 proxy, in the API Proxy editor (the new version).  Scroll the left-hand-side project navigator panel all the way down.  Select the runLoad.js file. This is the file that contains the generic logic for the load generator. You can browse the source of this nodejs module. It's about 36k of code. 
![runload-1 proxy](https://raw.githubusercontent.com/DinoChiesa/se-partner-bootcamp/master/loadgen/images/runload-1-proxy-runloadjs.png)

10. You don't need to understand all the code, but you should be aware of the design of runload.js, which is as follows:

  a. The runload nodejs server starts up and "listens" on a control interface. Via this REST interface you can send start or stop signals to the server, or you can query its status.  
  b. In the callback to the listen() function, runload calls setTimeout() to run a function "later".  In this case about 1200 ms later.  
  c. in the function called by the setTimeout(), runload reads a configuration file, then sends out a batch of API calls as described in that configuration file.   
  d. After receiving responses from the api calls, the runload logic then sets another timeout, for some delay. About 60 seconds by default, but this time varies depending on the configuration.   
  e. After the delay period, the timeout fires, and Javascript invokes the specified function. The effect is that runload "wakes up" and sends out another batch of API calls.   
  f. This continues indefinitely, or until you administratively "undeploy" the proxy, or send a "stop" command to the runload server. 
  
11. Now, back in the Edge UI, Select the model.json file in that lower panel.  
![runload-1 proxy](https://raw.githubusercontent.com/DinoChiesa/se-partner-bootcamp/master/loadgen/images/runload-1-proxy-modeljson.png)

12. This shows some basic configuration for runload. The json looks like this: 
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

13. Modify the configuration in model.json in this way: 

  a. set the "host" property to point to the DNS name of your Apigee Edge organization   
  b. Replace CLIENT_ID_HERE with the client_id you saved earlier  
  c. Likewise replace CLIENT_SECRET_HERE with the client_secret value  

14. Save the apiproxy. 

15. Deploy the API proxy. When runload deploys, it will read this configuration, and invoke the API call described in the model.json file, to send a request for a token, using client_credentials flow. 

16. In the Edge UI, go to the list of API Proxies, select the oauth proxy, and then select the Trace tab.  Start tracing this proxy. After a short delay you should see inbound requests arriving from loadgen / runload.js. If you do not, seek assistance from the lesson proctor. 

17. 
