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

_**NOTE:** for further reference please find out [this tutorial prepared by Dan van Leeuwen](https://developers.sap.com/tutorials/hana-cloud-alerts-custom.html), see these specific steps:_

_[Step 4 - Forward results to the SAP Alert Notification Service](https://developers.sap.com/tutorials/hana-cloud-alerts-custom.html#c514a77f-12af-499b-a639-1f5ef66e86c1) which describes
- How in SAP Alert Notification Service you can create a basic service key. Then, the service key will provide the URL and credentials for the SAP Automation Pilot to send notifications to the SAP Alert Notification Service.
- How to trigger a command in SAP Automaton Pilot with the input collected by SAP Alert Notfiicaiton service;
_
_[Step 5 - Send a notification when there are unassigned maintenance items](https://developers.sap.com/tutorials/hana-cloud-alerts-custom.html#3f5a1ba7-9ab1-45a5-a34c-f0a2235c8b57) - describing how to setup a new subsription in SAP Alert Notfiication service. 
_

**Automated action #1:** Collect data about long-running statment
We are going to use the SQL Active Statements in HANA Cloud so that SAP Automation Pilot is capable to get further details about the long-running statment by performing a query using the `ExecuteHanaCloudSqlStatement` command and provide further detaiils back to the Alert Notification service. 

The query DB will be based on this SQL Select: 
```
SELECT TOP 1000 DISTINCT  
	"CONNECTION_ID",
	"STATEMENT_ID",
	SUM("ALLOCATED_MEMORY_SIZE") AS "SUM_ALLOCATED_MEMORY_SIZE"
FROM "SYS"."M_ACTIVE_STATEMENTS"
GROUP BY "CONNECTION_ID", "STATEMENT_ID"
ORDER BY "CONNECTION_ID" ASC, "STATEMENT_ID" ASC

```
In order to achieve it we need to add an executor `SQLStatement` 
Provided command used: `sql-sapcp:ExecuteHanaCloudSqlStatement:1`

Command parameters:
connectionUrl —> `jdbc:sap://$(.scriptInput.host):$(.scriptInput.port)`
statement —> 
```
SELECT TOP 1000 DISTINCT  
	"CONNECTION_ID",
	"STATEMENT_ID",
	SUM("ALLOCATED_MEMORY_SIZE") AS "SUM_ALLOCATED_MEMORY_SIZE"
FROM "SYS"."M_ACTIVE_STATEMENTS"
GROUP BY "CONNECTION_ID", "STATEMENT_ID"
ORDER BY "CONNECTION_ID" ASC, "STATEMENT_ID" ASC
```
password —> `$(.scriptInput.password)`
resultRowFormat —> `OBJECT` 
resultTransformer —> `toArray[0]`
user —> `$(.scriptInput.user)` 

<br>![](/exercises/2/images/02_12_1.png)


Moreover, we need to add an executor `QueryDB` 
Provided command used: `scripts-sapcp:ExecuteScript:2`

script —> 
```
#!/usr/bin/env python3

import sys, json

input = sys.stdin.readline()
rows = json.loads(input)
print(rows)
```
stdin (sensitive string) —> `$(.SQLStatement.output.result)`

<br>![](/exercises/2/images/02_12_2.png)

Add Output Values —> `checkResult`
Configure checkResult string as it follows: `$(.QueryDB.output.output[0])` 

Run the command and within the output result you should be able to find results similar like this one: 
```
checkResult:[{'CONNECTION_ID': 205558, 'STATEMENT_ID': '882864887567552', 'SUM_ALLOCATED_MEMORY_SIZE': 7008}]
```


Add a new executor `NotifyANS`

Alias —> `NotifyANS`
Provided command —> monitoring-sapcp:SendAlertNotificationServiceEvent:1

Parameters added to the command: 

data —> 
```
{ "eventType": "HANA-AutoPiCustom", "severity": "WARNING", "category": "ALERT", "subject": "Alert Sent by SAP Automation Pilot", "body": "CONNECTION_ID: $(.SQLStatement.output.result|toArray[0]["CONNECTION_ID"]), STATEMENT_ID:$(.SQLStatement.output.result|toArray[0]["STATEMENT_ID"]), SUM_ALLOCATED_MEMORY_SIZE:$(.SQLStatement.output.result|toArray[0]["SUM_ALLOCATED_MEMORY_SIZE"])", "resource": { "resourceName": "HANA Cloud", "resourceType": "HANA Cloud Alerts" } }
```

password —> `$(.execution.input.ANSClientSecret)`

url —> https://clm-sl-ans-live-ans-service-api.cfapps.us10.hana.ondemand.com/cf/producer/v1/resource-events (NOTE: check your specific URL from the service key created within the Alert Notification service and add to it also the Producer API endpoint: ../cf/producer/v1/resource-events )

user —> $(.execution.input.ANSClientID)

The final command output should similar to this one: 

<br>![](/exercises/2/images/02_12_3.png)



_Expected result_

The outup (as a result) is further processed and forward to the Alert Notification service so the Ops Team can automatically collect the needed information needed for debugging and further investigations. See an example of an alert provided by the Alert Notification service to MS Team including data for debugging long-running statements. 
<br>![](/exercises/2/images/02_12_4.png)

**Automated action #2:** Disconnecting the session of a long-running statment
Following the same approach as described in Automated action #1 and already being able to identify the `CONNECTION_ID` for the long-running statement, the Automation Plot will execute an SQL statement to disconnect the related session, i.e. 
```
ALTER SYSTEM DISCONNECT SESSION '201939';

```

_Expected result_

The long running statment gets terminated automatically and there's no further system degradation caused on it. 
