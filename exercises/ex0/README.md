# Build a simple cloud application on SAP Business Applicaiton Studio

In this exercise, you will learn how to create an SAP Business Application Studio dev space, a preconfigured environment with the required tools and extensions tailored for a specific business scenario. Then you will find out how to create from a template a simple UI5 application, customise it, build the packaage and deploy it to your target CloudFoundry environment within an SAP BTP trial accout. 

## Create a Dev Space for SAP Fiori Apps and create your simple cloud UI5 application.

After completing these steps you will have a simple cloud app deployed on a CloudFoundry environment within an SAP BTP trial accout

1.	Access SAP Business Application Studio.
In a trial account, launch the SAP BTP trial landing page, and choose SAP Business Application Studio.
<br>![](/exercises/ex0/images/ex0_1.png)


2. In your trial account, access SAP BTP cockpit, navigate to the subaccount with SAP Business Application Studio subscription, click Subscriptions and Instances, and click SAP Business Application Studio.
<br>![](/exercises/ex0/images/ex0_2.png)

3. Enter `Demo_Fiori` for your dev space name. 

4. Then choose **SAP Fiori** as the application type . 

5. Click **Create Dev Space**.
<br>![](/exercises/ex0/images/ex0_4.png)
Then you shall see the dev space is in status `STARTING`. Wait until it is in status `RUNNING`. This might take a couple of minutes.

6. Open the SAP Fiori dev space by click the name of the dev space you just have created.
<br>![](/exercises/ex0/images/ex0_5.png)

7. The SAP Fiori dev space opens and the Welcome tab appears.
<br>![](/exercises/ex0/images/ex0_6.png)

8. Connect to a Cloud Foundry endpoint
You will need to log on to Cloud Foundry. To do that, start the command palette from the menu **View > Find Command**, search for `cf:login`, and select the command `CF:Login to Cloud Foundry`.
<br>![](/exercises/ex0/images/ex0_7.png)

Then, you shall enter the Cloud Foundry endpoint you want to use. Enter your email and your password to proceed.
<br>![](/exercises/ex0/images/ex0_8.png)

Select the Cloud Foundry Organization and space you want use. You will see messages in the lower right hand corner indicating that you are connected to the endpoint once these prompts have been answered.
<br>![](/exercises/ex0/images/ex0_9.png)

9. Create your simple UI5 Cloud app 
SAP Fiori tools includes an Application Generator that provides a wizard-style approach for creating applications.

To launch the Application Generator, start the command palette from the menu item View > Find Command, search for `fiori generator`, and select the command Fiori: Open Application Generator

Specify the application type SAPUI5 freestyle and the floor plan `SAPUI5 Application` and go to the **Next** screen.
<br>![](/exercises/ex0/images/ex0_10.png)

Specify the Data Source as `None` and click on the **Next** button. 
<br>![](/exercises/ex0/images/ex0_11.png)
_**NOTE** Considering this is going to be a static UI5 app - we won't pick up a Data Source at that point. _

Keep `View1` within the **View name** entity selection field and click on the **Next** button. 
<br>![](/exercises/ex0/images/ex0_12.png)

Withi the Project Attributtes configuration please fill in the data as shown below and click on the **Finish** button. 
<br>![](/exercises/ex0/images/ex0_13.png)

Then, you will see that dependencies are in a process of installation
<br>![](/exercises/ex0/images/ex0_14.png)

Once, created , we can see the codebase for our application. 
<br>![](/exercises/ex0/images/ex0_15.png)

9. Modify the homepage of your application 
We can modify the homepage of our application by simply using the SAP Business Application Studio built-in Layout Editor feature. In order to do so, please navigatre to your app view at `View1.view.xml` 
<br>![](/exercises/ex0/images/ex0_16.png)

Open the view with the  Layout Editor. 
<br>![](/exercises/ex0/images/ex0_17.png)

By using the available controls we can add the following elements just to customise the homepage of our app: 
- icon 
- text area 
<br>![](/exercises/ex0/images/ex0_18.png)

10. Add deplopyment configuration 
Click on the Run task ... , then find and execute the command: `npm: deploy-config`
<br>![](/exercises/ex0/images/ex0_19.png)

Within the Terminal , choose that you want to add a deployment configrarion for a `CloudFoundry` setup and for destination you can pick-up `None` and add the application to a `managed application router`
<br>![](/exercises/ex0/images/ex0_20.png)

At executing the command you will see that the following assets have been created: 
`Updating mta.yaml with module information
   create xs-security.json
   create xs-app.json
   create ui5-deploy.yaml` 

11. Build the project 
Build (aka package) the project to a mtar archive to deploy it to Cloud Foundry. Right-click on the `mta.yaml` file and select Build MTA Project to trigger this process.
<br>![](/exercises/ex0/images/ex0_21.png)

Once the build is complete, you can see a message in the log. You can now find the generated `mtar` archive in the project tree under `mta_archives`.
<br>![](/exercises/ex0/images/ex0_22.png)

12. Deploy your project 
Access the Applicaiton info page: from the Command Palette from menu **View > Find Commands**, type Application Info, and select `Fiori: Open Application Info`. Click on the **Deploy Application** tile to start deployment process
<br>![](/exercises/ex0/images/ex0_23.png)

The deployment process has been just started.
<br>![](/exercises/ex0/images/ex0_24.png)

The deployment process has been  finished.
<br>![](/exercises/ex0/images/ex0_25.png)

13. Access your application over BTP cockipt 
To do so , navigate to your BTP Trial sub-account and click on the HMTL5 Applicaiton menu. Then click on the application you just have deployed.
<br>![](/exercises/ex0/images/ex0_26.png)

Well done - you will be landing to your app home page!
<br>![](/exercises/ex0/images/ex0_27.png)


## Summary

Now that you have the cloud application in place we can proceed to the next step.
Continue to - [Exercise 2 - Cloud Application Monitoring](../ex1/README.md)
