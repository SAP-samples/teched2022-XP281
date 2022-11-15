# Exercise 1 - Description

Within this execercise we are going to cover a potential use case of a development team that has developed an SAPUI5 Web App deployed on SAP BTP Cloud Foundry. However, sometimes there are disruptions / outages with their SAPUI5 Web App. The app is consumed through the SAP Launchpad service providing a central entry point with an easy and efficient access to relevant applications for the business users. Nevertheless, for a certain duration the app becomes inaccessible randomly for different users and as of now there are not any alerts sent out.   

## Main goal
By using monitoring and automation services within the SAP BTP DevOps portfolio, we should be capable to build an integrated monitoring and health checks solution for the SAP UI5 app, so that in case of a disruption there is an alert fired immediately.  

## Additional preparation
(covered also during the workshop) 

**IMPORTANT**: For the purpose of this excercise we will use the Cloud UI5 Application already created in the previous excercise, please see: [Exercise 1 - Build a Cloud UI5 Applicaiton on SAP Business Application Studio](../ex2/README.md)

1. Configure your global account and subaccount in BTP 
2. [Setup Alert Notification service](https://help.sap.com/docs/ALERT_NOTIFICATION/5967a369d4b74f7a9c2b91f5df8e6ab6/812b6e3ed8934648ad15780cd51721ef.html) 
3. [Setup Automation Pilot ](https://help.sap.com/docs/AUTOMATION_PILOT/de3900c419f5492a8802274c17e07049/76e77c4563d042b2b46f6c622be3a091.html)

## What you will learn
**1. Learn the basics of SAP BTP** 
SAP Business Technology Platform (SAP BTP) offers you the ability to turn data into business value, compose end-to-end business processes, and build and extend existing SAP solutions quickly. 
Before starting with developing extension applications, you need to understand how SAP BTP is structured so you can choose the best components for your use case. Take a look at Cloud Foundry and Kyma environments, global accounts and subaccounts.  
 
These are the best resources to start from: 
- [SAP BTP Product Page ](https://help.sap.com/viewer/product/CP/Cloud/en-US?task=discover_task)
- [What Is SAP BTP](https://help.sap.com/viewer/3504ec5ef16548778610c7e89cc0eac3/Cloud/en-US/73beb06e127f4e47b849aa95344aabe1.html)
- [Cloud Foundry Environment](https://help.sap.com/viewer/3504ec5ef16548778610c7e89cc0eac3/Cloud/en-US/9c7092c7b7ae4d49bc8ae35fdd0e0b18.html)

**2. Learn the basics of SAP Alert Notification service for SAP BTP**
SAP Alert Notification service for SAP BTP is a cloud service offering in SAP Business Technology Platform (BTP) DevOps portfolio. It is a real-time alerting engine which is designed to collect alerts from different providers. Customers can subscribe to the alerts of their interest and choose the most appropriate channel for delivery. The service masters in collecting crucial technical information from the SAP BTP service, also it can handle information regarding any custom scenario within your own environment.  
<br>![](/exercises/1/images/01_01_00.png)

**3. Learn the basics of SAP Automation Pilot**
The primary goal of SAP Automation Pilot is to simplify and automate complex manual processes in order to minimize the operational effort behind any cloud solution in the SAP Business Technology Platform. By using the means of its minimalistic model, you can automate a sequence of steps executable by a machine. The service is designed to work with low latency even when it is under a heavy workload. 
<br>![](/exercises/1/images/01_01_01.png)

## Exercise: Concept

The solution to be built is based on a synergy and native integration between the SAP Alert Notification Service and  SAP Automation Pilot as integrating their existing capabilities for Ops automation and Alerting in SAP BTP we can build the needed SAPUI5 Web App health check monitoring.  

The architecture design covers the assumptions as listed below:  
- There is an application health-check endpoint that to be called regularly  
- SAP Automation Pilot is to be automation engine that triggers periodically health-check status calls to the app health-check end points  
- Based on the responses received by the SAP Automation Pilot, there will be a condition within the SAP Automation Pilot so that:  
  - If the http response status code is “200” – then the application is considered accessible  
  - If the http response status code is different than “200” – the application is considered not to be accessible an alert is to be sent out to the DevOps team  

The expected use case (positive and negative application responses) can be visualized as shown underneath:  

**SAPUI5 Web App is UP & RUNNING **
<br>![](/exercises/1/images/01_01.png)
Diagram explained:  
1. There is a command build via the SAP Automation Pilot UI which calls the application health-check endpoint each 5 minutes.  
2. The application health-check endpoint returns http response status code: “200” 
3. Based on a condition set within the SAP Automation Pilot command, if the response is “200” there is no need of further actions.  
 
**SAPUI5 Web App  is NOT accessible  **
<br>![](/exercises/1/images/01_02.png)

Diagram explained:   
1. There is a command build via the SAP Automation Pilot UI which calls the application health-check endpoint each 5 minutes.  
2. The application health-check endpoint returns http response status code that is NOT “200” 
3. Based on a [condition set within the SAP Automation Pilot command](https://help.sap.com/docs/AUTOMATION_PILOT/de3900c419f5492a8802274c17e07049/2d754542d0914e218d79719f7bd9600f.html), if the response is different than “200” then the SAP Automation Pilot should notify the SAP Alert Notification service.  there is a [custom event to be produced by the SAP Automation Pilot which is to be consumed by the SAP Alert Notification Service](https://help.sap.com/docs/AUTOMATION_PILOT/de3900c419f5492a8802274c17e07049/6124b87d6e0249be9ffbc5a091123f97.html).
4. In the SAP Alert Notification Service there is a subscription which is configured to filter events received by the service so that in case there is an event which falls into [predefined conditions](https://help.sap.com/docs/ALERT_NOTIFICATION/5967a369d4b74f7a9c2b91f5df8e6ab6/35ca5de101fc4d5791cdbb2df15e9d9b.html), a [specific action](https://help.sap.com/docs/ALERT_NOTIFICATION/5967a369d4b74f7a9c2b91f5df8e6ab6/8a7e092eebc74b3ea01d506265e8c8f8.html) is to be fired - in our case that is an email to be sent out to the DevOps team.  


# Exercise 1 - Implementation

**1. Cloud UI5 app health-check endpoint**
- Within the already existing Cloud UI5 app please we need to setup a health-check endpoint. You can build the endpoint as an simple html file which is to be called  
<br>![](/exercises/1/images/01_03.png)

- In order  to call the specific html file from Automation Pilot perspective you should include the following settings within the `xs-app.json` file into your SAP UI5 app
<br>![](/exercises/1/images/01_04.png)

**2. SAP Automation Pilot health-check command**  
- For set up a health-check command please reuse  the command [HttpRequest already provided by the SAP Automation Pilot](https://help.sap.com/docs/AUTOMATION_PILOT/de3900c419f5492a8802274c17e07049/6ce1e04b7812411db04b80ea769ef46e.html), where you can modify the method and the URL.  
<br>![](/exercises/1/images/01_05.png)

- Conditions within the SAP Automation Pilot which will fire the custom event
<br>![](/exercises/1/images/01_06.png)

- The data object for the custom event that is to be produced by the SAP Automation Pilot in case the condition is met:  

`{ "eventType": "CUSTOM-ALERT", "severity": "WARNING", "category": "ALERT", "subject": "Alert Sent by SAP Automation Pilot: SAPUI5 Web App Is Not Accessible", "body": "IMPORTANT! SAPUI5 Web App cannot be accessed by the Automation Pilot health check request.", "resource": { "resourceName": "App #2", "resourceType": "application" } } `

- The regular and repetitive health-checks calls are performed via the Automation Pilot built-in [job scheduler](https://help.sap.com/docs/AUTOMATION_PILOT/de3900c419f5492a8802274c17e07049/96863a2380d24ba4bab0145bbd78e411.html), currently set to execute the command in question each 5 mins.  
<br>![](/exercises/1/images/01_07.png)

**3. SAP Alert Notification service subscription setup**
- Condition is setup based on the event type received as shown underneath:   
<br>![](/exercises/1/images/01_08.png)

- The expected action is configured to fire an alert email:
<br>![](/exercises/1/images/01_09.png)

**4. Expected result**
After completing these steps you will have created a health status check solution within SAP BTP covering: 
- the app health check commands are fired regularly checking the Web App accessibility 
- in case of an issue identified there is an email sent out to the DevOps team as per the example shared below 
<br>![](/exercises/1/images/01_10.png)

...

## Summary
You've now have a simple app running on BTP CloudFoundry for which we have a health check monitoring and alerting. 

Continue to - [Exercise 3 - HANA Cloud Monitoring and Automated Remediations](../2/README.md)
