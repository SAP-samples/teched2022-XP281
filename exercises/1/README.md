# Exercise 2 - Description

Within this execercise we are going to cover a potential use case of a development team that has developed an SAPUI5 Web App deployed on SAP BTP Cloud Foundry. The app is consumed through the SAP Launchpad service providing a central entry point with an easy and efficient access to relevant applications for the business users. Nevertheless, if the app becomes randomly inaccessible for a certain duration, as of now there are not alerts of that unavailability sent out.   

## Main goal
By using monitoring and automation services within the SAP BTP DevOps portfolio, we should be capable to build an integrated monitoring and health checks solution for the SAP UI5 app, so that in case of a disruption there is an alert fired immediately.  

## Additional preparation
(covered also during the workshop) 

**IMPORTANT**: For the purpose of this excercise we will use the Cloud UI5 Application already created in the previous excercise, please see: [Exercise 1 - Build a Cloud UI5 Applicaiton on SAP Business Application Studio](../2/README.md)

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
- [Kyma Environment](https://help.sap.com/docs/BTP/65de2977205c403bbc107264b8eccf4b/468c2f3c3ca24c2c8497ef9f83154c44.html)

**2. Learn the basics of SAP Alert Notification service for SAP BTP**
SAP Alert Notification service for SAP BTP is a cloud service offering in SAP Business Technology Platform (BTP) DevOps portfolio. It is a real-time alerting engine which is designed to collect alerts from different providers. Customers can subscribe to the alerts of their interest and choose the most appropriate channel for delivery. The service masters in collecting crucial technical information from the SAP BTP service, also it can handle information regarding any custom scenario within your own environment.  
<br>![](/exercises/1/images/01_01_00.png)

**3. Learn the basics of SAP Automation Pilot**
The primary goal of SAP Automation Pilot is to simplify and automate complex manual processes in order to minimize the operational effort behind any cloud solution in the SAP Business Technology Platform. By using the means of its minimalistic model, you can automate a sequence of steps executable by a machine. The service is designed to work with low latency even when it is under a heavy workload. 
<br>![](/exercises/1/images/01_01_01.png)

## Exercise: Concept

The solution to be built is based on a synergy and native integration between the SAP Alert Notification Service and  SAP Automation Pilot as integrating their existing capabilities for Ops automation and Alerting in SAP BTP we can build the needed SAPUI5 Web App health check monitoring. Using those powerful primitives of those services, interaction with complex app API and real-user-like monitoring can be easily achieved just as well.

The architecture design covers the assumptions as listed below:  
- There is an application health-check endpoint that to be called regularly  
- SAP Automation Pilot is to be automation engine that triggers periodically health-check status calls to the app health-check end points  
- Based on the responses received by the SAP Automation Pilot, there will be a condition within the SAP Automation Pilot so that:  
  - If the http response status code is “200” – then the application is considered accessible  
  - If the http response status code is different than “200” – the application is considered not to be accessible an alert is to be sent out to the DevOps team  

The expected use case (positive and negative application responses) can be visualized as shown underneath:  

**SAPUI5 Web App is UP & RUNNING**
<br>![](/exercises/1/images/01_01.png)
Diagram explained:  
1. There is a command build via the SAP Automation Pilot UI which calls the application health-check endpoint each 5 minutes.  
2. The application health-check endpoint returns http response status code: “200” 
3. Based on a condition set within the SAP Automation Pilot command, if the response is “200” there is no need of further actions.  
 
**SAPUI5 Web App  is NOT accessible**
<br>![](/exercises/1/images/01_02.png)

Diagram explained:   
1. There is a command build via the SAP Automation Pilot UI which calls the application health-check endpoint each 5 minutes.  
2. The application health-check endpoint returns http response status code that is NOT “200” 
3. Based on a [condition set within the SAP Automation Pilot command](https://help.sap.com/docs/AUTOMATION_PILOT/de3900c419f5492a8802274c17e07049/2d754542d0914e218d79719f7bd9600f.html), if the response is different than “200” then the SAP Automation Pilot should notify the SAP Alert Notification service.  there is a [custom event to be produced by the SAP Automation Pilot which is to be consumed by the SAP Alert Notification Service](https://help.sap.com/docs/AUTOMATION_PILOT/de3900c419f5492a8802274c17e07049/6124b87d6e0249be9ffbc5a091123f97.html).
4. In the SAP Alert Notification Service there is a subscription which is configured to filter events received by the service so that in case there is an event which falls into [predefined conditions](https://help.sap.com/docs/ALERT_NOTIFICATION/5967a369d4b74f7a9c2b91f5df8e6ab6/35ca5de101fc4d5791cdbb2df15e9d9b.html), a [specific action](https://help.sap.com/docs/ALERT_NOTIFICATION/5967a369d4b74f7a9c2b91f5df8e6ab6/8a7e092eebc74b3ea01d506265e8c8f8.html) is to be fired - in our case that is an email to be sent out to the DevOps team.  


# Exercise 1 - Implementation
**1. Cloud UI5 app health-check endpoint**

- Within the already existing Cloud UI5 app please we need to setup a health-check endpoint. You can build the endpoint as a simple html file which is to be named `health-check.html`  
<br>![](/exercises/1/images/01_03.png)
NOTE: adding the html code underneath as well:
```html
<!DOCTYPE html>
<html>
    <body>
        <p><b>Welcome to our SAP UI5 Web App Healthcheck end-point! </b></p>
        <p><b>If you read this message - your application is live. </b></p>

    </body>
</html>
```
- In order to query the specific html file without authorization, you should include the following settings within the `xs-app.json` file into your SAP UI5 app.
<br>![](/exercises/1/images/01_04.png)
```json
{
  "source": "^/health-check.html$",
  "service": "html5-app-repo-rt",
  "httpMethods": ["GET","POST"],
  "authenticationType": "none"
}
```

- Build and redeploy the application via the SAP Business Applicaiton Studio (same steps as the ones for build app package and deployment described in Excercise #1) 

- Check the health-check endpoint it is accessible for an anonymous users. 
<br>![](/exercises/1/images/01_04_1.png)

**2. Setup Alert Notification**
- To setup Alert Notification service for SAP BTP, go to the BTP cockpit, navigate to the subaccount, `Cloud Foundry` -> `Spaces` tab and enter your space.
- Navigate to the `Marketplace` and on the `Alert Notification` card menu, coose `create`. Give it a name, default configurations will do.
<br>![](/exercises/1/images/01_04_1_1.png)
- In the `Instances` tab in the same space you'll find the service newly created instance. For it, create a service key with which the service API will be accessed.
<br>NOTE: For the Configuration Json parameters use: 
  ```json
  {
    "type": "BASIC"
  }
  ```
  <br>![](/exercises/1/images/01_04_2.png)
- From here, you're able to view the service key credentials and use them to grant other services access to this particular ANS instance, e.g. the Automation Pilot.
<br>Keep them for a moment as we will need the credentials soon. 
<br>![](/exercises/1/images/01_04_3.png)
                

**3. Setup SAP Automation Pilot**

- Navigate to your subaccount's `Service Marketplace` tab and choose `Create` on Automation Pilot's cart to subscribe to the service
<br>![](/exercises/1/images/01_04_4.png)

- Navigate to the `Security`, `Users` tab and assing the `AutomationPilot_Admin` role collection to your user.
<br>![](/exercises/1/images/01_04_5.png)

- In the `Instances & Subscriptions` tab in your subaccount, open your Automation Pilot application subscription
<br>![](/exercises/1/images/01_04_6.png)

- Create an Service Account for the Automaiton Pilot and give the needed permissions as per the screenshot. 
<br>NOTE: Set `Authentication Type` to `Basic`. 
<br>![](/exercises/1/images/01_05_1.png)

- Keep the credentials of your Service Account as it won't be available later but it will be required when creating input data within the SAP Automation Pilot. 
<br>![](/exercises/1/images/01_05_2.png)

**4. Model the health-check command in SAP Automation Pilot**  

- In the `Own` Catalog page, create a new catalog to hold the health check commands and relevant inputs
<br>![](/exercises/1/images/01_05_0.png)

- Create an input `AutoPiSecret` within the Inputs section in Automation Pilot where we will add the Automaution Pilot password that we just have created and stored from the previous step. 
<br>![](/exercises/1/images/01_05_3.png)

- Create an input `cloudAppInputs` within the Inputs section in Automation Pilot where is going to be specified
  -  `appMethod`: GET
  -  `endPoint`: the full URL for the health-check.html file we have created 
<br>For reference, see the screenshot: 
<br>![](/exercises/1/images/01_05_4.png)

- Create the needed inputs for the Alert Notification service - named `ANSUserInput`.

  -> client_id -> copy value from service key created within the Alert Notification service

  -> client_secret -> (sensitive data) copy value from service key created within the Alert Notification service
<br><br>The final result should look like this one:
<br>![](/exercises/1/images/01_05_12.png)

- To set up a health-check command you can use the command [HttpRequest already provided by the SAP Automation Pilot](https://help.sap.com/docs/AUTOMATION_PILOT/de3900c419f5492a8802274c17e07049/6ce1e04b7812411db04b80ea769ef46e.html), where you can modify the method and the URL.  
<br>![](/exercises/1/images/01_05.png)
<br>To do so, access the command provided within the respective Provided Catalog by the SAP Automation Pilot - `HTTP Operations (http-sapcp)`, open it and then click on the **"Clone"** button. You can give a meaningful name to the newly created command, i.e. `AppHealhCheck`. 

- Within the **Input Keys** section use the input keys that you have created in the previous steps and specify the input keys as it follows: 
  - `methond`
  - `url`
  - `ANSClientID` --> Default Value: key client_id from input ANSUserInput (already created) 
  - ANSClientSecret --> Default Value: key client_secret from input ANSUserInput (already created) 

<br>You can remove the other Input Keys so you are left just with these four: 
<br>![](/exercises/1/images/01_05_5_1.png)

- Remove the Output Keys and keep just the one for `status`. 
<br>![](/exercises/1/images/01_05_7.png)

- Add an **Executor** for the the command you just have cretead and use a command `http-sapcp:HttpRequest:1` and name this step `appHealthCheck`

  Keep the paramenters within the command as specified here: 
<br>![](/exercises/1/images/01_05_8.png)

- Access the **output** and change the Output Values for `status` (number) to: `$(.appHealthCheck.output.status)`
<br>![](/exercises/1/images/01_05_9.png)

- Add a new executor `notifyANS` of the `monitoring-sapcp:SendAlertNotificationServiceEvent:1` command which would send the notification event
<br>As we see the command expect to specify parameters for: 
  - data: the notification event payload
  - password: this is the `ANSClientSecret` already created in the Input Keys
  - url: endpoint of the ANS service which is a value from ANS service key + `/cf/producer/v1/resource-events`
  - user: this is the `ANSClientID` already created in the Input Keys
  - tokenURL (optional): used for other auth methods --> we can leave it empty

 NOTE: `data` object
 The data object for the custom event that is to be produced by the SAP Automation Pilot in case the condition is met:  
    ```json 
    { "eventType": "CUSTOM-ALERT", "severity": "WARNING", "category": "ALERT", "subject": "Alert Sent by SAP Automation Pilot: SAPUI5 Web App Is Not Accessible", "body": "IMPORTANT! SAPUI5 Web App cannot be accessed by the Automation Pilot health check request.", "resource": { "resourceName": "cloudApp", "resourceType": "application" } }
    ```
  <br>  ![](/exercises/1/images/01_05_13_0.png)
  <br>  ![](/exercises/1/images/01_05_13_1.png)

- Condition of that executor to fire only when the response from our app was not 200
  <br>  ![](/exercises/1/images/01_05_13_2.png)

  Resulting executor should like this: 
<br>![](/exercises/1/images/01_05_13.png)

- Scheduled, repetitive or one-time command executions can be set up via the SAP Automation Pilot's built-in [job scheduler](https://help.sap.com/docs/AUTOMATION_PILOT/de3900c419f5492a8802274c17e07049/96863a2380d24ba4bab0145bbd78e411.html). Set a schedule to execute the command in question each 5 minutes.  
<br>![](/exercises/1/images/01_07.png)

**3. SAP Alert Notification service notification subscription**
<br>Create a new subscriptions for the notifications fired by our scheduled Automation Pilot command
- Setup a condition, based on the event type received as shown underneath:   
<br>![](/exercises/1/images/01_08.png)

- The expected action is configured to send email
Note: if this is the first time you'd be receiving email notifications via ANS you'd receive an email to confirm such deliveries first.
<br>![](/exercises/1/images/01_09.png)

**4. Expected result**
After completing these steps you will have created a health status check solution within SAP BTP covering: 
- the app health check commands are fired regularly checking the Web App accessibility 
- in case of an issue identified there is an email sent out to the DevOps team as per the example shared below 
<br>![](/exercises/1/images/01_10.png)

...

## Summary
You've now have a simple app running on BTP CloudFoundry for which we have a health check monitoring and alerting. 

Continue to - [Exercise 3 - HANA Cloud Monitoring and Automated Remediations](../3/README.md)

