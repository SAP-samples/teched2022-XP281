# Exercise 3 - HANA Cloud monitoring and automated remediation:  Description

In this exercise, we will create Hana Cloud queries using the provided ExecuteHanaCloudSqlStatement and also we will conditionally send a notification (email) with the results of the query using the SAP Alert Notification Service. 

# Prerequisits
You have covered the following tutorials: 
- [SAP HANA Database Explorer Overview]([url](https://developers.sap.com/tutorials/hana-dbx-overview.html))
- [Add Databases to the SAP HANA Database Explorer]([url](https://developers.sap.com/tutorials/hana-dbx-connections.html))
- Setup a HANA Cloud within your trial BTP account (this is to be covered also during the workshop itself)

# Implementation

A SAP HANA Cloud database or a data lake Relational Engine instance have a set of built-in alerts that when triggered, are sent to the SAP Alert Notification Service (ANS). This service, in addition to being able to forward details of the alert to various channels (email, Microsoft Teams, Slack, etc.), can trigger a SAP Automation Pilot command. 
<br>![](/exercises/ex2/images/02_01.png)

For this worksho we'll receive an alert from the Alert Notification service about `HDB Long Running Statement`
<br>![](/exercises/ex2/images/02_02.png)

Further details / specifics on the alert itself can be found at the Alert Notification service help documentation here: [HDB Long Running Statement](https://help.sap.com/docs/ALERT_NOTIFICATION/5967a369d4b74f7a9c2b91f5df8e6ab6/0bf53562d11548abb66a01444a25b070.html). 


Then the Automation Pilot is to get further details from the HANA Cloud by execute SQL Commandsand (by `QueryDB`)  and provide further detaiils back to the Alert Notification service. 

<br>![](/exercises/ex2/images/02_03.png)

<br>![](/exercises/ex2/images/02_04.png)
