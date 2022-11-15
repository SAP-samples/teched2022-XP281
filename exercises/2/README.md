# Exercise 3 - HANA Cloud monitoring and automated remediation:  Description

In this exercise, we will create HANA Cloud queries using the provided ExecuteHanaCloudSqlStatement and also we will conditionally send a notification (email) with the results of the query using the SAP Alert Notification Service. 

# Prerequisits
You have covered the following tutorials: 
- [SAP HANA Database Explorer Overview]([url](https://developers.sap.com/tutorials/hana-dbx-overview.html))
- [Add Databases to the SAP HANA Database Explorer]([url](https://developers.sap.com/tutorials/hana-dbx-connections.html))
- Setup a HANA Cloud within your trial BTP account (this is to be covered also during the workshop itself)

# Implementation

## Triggering an alert from HANA Cloud

A SAP HANA Cloud database or a data lake Relational Engine instance have a set of built-in alerts that when triggered, are sent to the SAP Alert Notification Service (ANS). This service, in addition to being able to forward details of the alert to various channels (email, Microsoft Teams, Slack, etc.), can trigger a SAP Automation Pilot command. 
<br>![](/exercises/2/images/02_01.png)

For this workshop we are going to receive an alert fired from the Alert Notification service based on a logic to consume an event initiated by the HANA Cloud about `HDB Long Running Statement`.
<br>![](/exercises/2/images/02_02.png)

Further details / specifics on the alert itself can be found at the Alert Notification service help documentation here: [HDB Long Running Statement](https://help.sap.com/docs/ALERT_NOTIFICATION/5967a369d4b74f7a9c2b91f5df8e6ab6/0bf53562d11548abb66a01444a25b070.html). 

In order to simulated such an event / sitation from the HANA Cloud side, we will need to cover couple of actions: 

1. Set the needed alert definitions within the HANA Cloud 
Open the SAP HANA Cockpit --> In the Monitoring view, open the Alert Definitions app -->  Then look for this alert as shown on the screenshot below: 
<br>![](/exercises/2/images/02_06.png)

Then click on the alert and modify the intervals for checks and Thresholds as show below (this is needed in order to prioritise and detect as early as possible the long-running statements considering the limited workshop duration). 
<br>![](/exercises/2/images/02_07.png)

Once the changes are applied, just click on the "Save" button

2. Trigger the alerts in SAP HANA Database 
Open the SAP HANA database explorer --> Execute the following SLQ commands within the SQL Console accessible through the SAP HANA Database Explorer
<br>![](/exercises/2/images/02_05.png)

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

3. As a result there will be an alert logged into the the Database Overview in the SAP HANA cockpit indicating an alert that was triggered. 
<br>![](/exercises/2/images/02_08.png)

## Alert Notification service: trigger an alert and deliver it to a DevOps 

The Alert Notification service has to be configured to act on the incoming notifications by sending an email with the details of the alert. To do so, couple of notes should be followed: 

1. Access the Alert Notification service instance through the SAP BTP Cockpit. 
NOTE: The Alert Notification Service must be in the same cloud foundry subaccount as the SAP HANA Cloud instances that it will be receiving notifications from.

2. Create a condition 
<br>![](/exercises/2/images/02_09.png)

3. Create an action (we are going to use a simple email action type) 
<br>![](/exercises/2/images/02_10.png)

4. Create a subscription attaching to it the condition and the action alreawy created. 
<br>![](/exercises/2/images/02_11.png)

**Expected result:**

When there is an alert coming from HANA Cloud, it will be detected from the Alert Notification service and an alert should be sent out to the Ops Team, including the alert information as shown on the email underneath. 
<br>![](/exercises/2/images/02_12.png)



## Automation Pilot: Remedition actions 

Within this use case we will onboard SAP Automation Pilot to automated possible remediation action in case of a specific alert related to HANA Cloud gets detected by the Alert Notification service. 

**How?**

There will be two possible automated actions initiated by the Automation Pilot. Both of them are based on features to execute SQL Commands and Create Custom Notifications with SAP Automation Pilot and SAP Alert Notification Service . 

_**NOTE:** for further reference please find out [this tutorial prepared by Dan van Leeuwen](https://developers.sap.com/tutorials/hana-cloud-alerts-custom.html)._

**Automated action #1:** Collect data about long-running statment
We are going to use the SQL Plan Cach in HANA Cloud so that SAP Automation Pilot is to get further details about the long-running statment by performing a query using the `ExecuteHanaCloudSqlStatement` command and provide further detaiils back to the Alert Notification service. 

The query DB will be based on the `STATEMENT_HASH` and shall look like this one: 
```
SELECT *
FROM M_SQL_PLAN_CACHE
WHERE STATEMENT_HASH = 'bcd01a3e39ba4a82758ea300fd8d5a9a';

```
NOTE: The `STATEMENT_HASH` is known to the Automation Pilot as it is available within the alert sent by the Alert Notification service, i.e. 

```
Event Body:
Identifies long-running SQL statements.
This statement has been running for 281 seconds, Statment Hash: bcd01a3e39ba4a82758ea300fd8d5a9a, Transaction ID: 315, client PID: 170, thread ID: 3554, statement ID: 890320950785270.
```
<br>![](/exercises/2/images/02_13.png)

Plus we can specify the values for the SQL Statement: 
<br>![](/exercises/2/images/02_14.png)

_Expected result_

The outup (as a result) is further processed and forward to the Alert Notification service so the Ops Team can automatically collect the needed information needed for debugging and further investigations.


**Automated action #2:** Disconnecting the session of a long-running statment
Following the same approach as described in Automated action #1 and already being able to identify the `CONNECTION_ID` for the long-running statement, the Automation Plot will execute an SQL statement to disconnect the related session, i.e. 
```
ALTER SYSTEM DISCONNECT SESSION '201939';

```

_Expected result_

The long running statment gets terminated automatically and there's no further system degradation caused on it. 