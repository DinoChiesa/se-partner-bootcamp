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

1. Deploy the oauth proxy.   You may need to import the oauth proxy; if it is already imported into your organization, verify it is deployed to the test environment, and if not, actually deploy it to test.   The oauth proxy will dispense tokens for other APIs in use in your organization.

  a. Open up a browser tab and log in to http://edge.apigee.com
  b. From the Organization drop-down in the top-right corner, select the organization assigned to you
  c. From the Environment drop-down, select "test"
  d. From the main menu, select APIs ? API Proxies

