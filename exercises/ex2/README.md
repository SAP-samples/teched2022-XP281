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

For this workshop we are going to receive an alert fired from the Alert Notification service based on a logic to consume an event initiated by the HANA Cloud about `HDB Long Running Statement`.
<br>![](/exercises/ex2/images/02_02.png)

Further details / specifics on the alert itself can be found at the Alert Notification service help documentation here: [HDB Long Running Statement](https://help.sap.com/docs/ALERT_NOTIFICATION/5967a369d4b74f7a9c2b91f5df8e6ab6/0bf53562d11548abb66a01444a25b070.html). 

In order to simulated such an event / sitation from the HANA Cloud side, we will need to execute the following SLQ commands within the SQL Console accessible through the SAP HANA Database Explorer
<br>![](/exercises/ex2/images/02_05.png)

Sharing the command iself for further usage: 
```
DO BEGIN
  -- Wait for a few seconds
  USING SQLSCRIPT_SYNC AS SYNCLIB;
  CALL SYNCLIB:SLEEP_SECONDS( 300 );  --runs for 5 minutes
  -- Now execute a query
  SELECT * FROM M_TABLES;
END;
```

Then the Automation Pilot is to get further details from the HANA Cloud by execute SQL Commandsand (by `QueryDB`)  and provide further detaiils back to the Alert Notification service. 

<br>![](/exercises/ex2/images/02_03.png)

<br>![](/exercises/ex2/images/02_04.png)
