[![REUSE status](https://api.reuse.software/badge/github.com/SAP-samples/teched2022-XP281)](https://api.reuse.software/info/github.com/SAP-samples/teched2022-XP281)
# XP281 - Monitoring and Ops Automation for Your Cloud Application and SAP HANA Cloud

## Description

This repository contains the material for the SAP TechEd 2022 session called Session XP281 - Monitoring and Ops Automation for Your Cloud Application and SAP HANA Cloud 

## Overview

This session introduces attendees to some of the operations products within the SAP Business Technology Platform (BTP) porfolio such as: 
- SAP Alert Notification service for SAP BTP 
- SAP Automation Pilot 

The main purpose of this workshop is to provide more insights and learn how to combine alerting capabilities from SAP Alert Notification service for SAP BTP with automation of ops tasks from SAP Automation Pilot service in order to cover two main use cases: 
- business check about the application health status 
- monitoring and automated remediation for your instance of SAP HANA Cloud

## Requirements

The requirements to follow the exercises in this repository are: 
- You have access to SAP Business Application Studio (to onboard SAP Business Application Studio in a trial account, see [Set Up SAP Business Application Studio for Development](https://developers.sap.com/tutorials/appstudio-onboarding.html)).
- Access to the SAP Business Technology Platform (BTP) that includes SAP HANA Cloud, SAP Alert Notification Service, and SAP Automation Pilot. These services are available in the SAP BTP trial accounts and have a free plan.

## Exercises

During the workshop itself there will be two main excercises related to the monitoring and automated operatins: one covering a cloud application and another one with a focus towards HANA Cloud. Furher details are provided by linking to the specific exercise pages as it follows:

- [Exercise 1 - Build a Cloud UI5 Application on SAP Business Applicaiton Studio](exercises/0/)
- [Exercise 2 - Cloud Application Monitoring with Alert Notificaiton service and Automation Pilot](exercises/1/)
- [Exercise 3 - HANA Cloud monitoring and automated remediation](exercises/2/)

Note: If executing exercies in an SAP BTP Trial account with available subaccount regions US East (`us10`) and Singapore (`ap21`) 
- Exercise 1 can be done both in subaccounts of regions us10 and and ap21
- Exercise 2, the subscription of Automation Pilot should be done in an ap21 subaccount, and would be cross-consumed to operate over us10 as well as ap21.
- Exercise 3, the subscriptions of Hana Cloud and Alert Notification should both be done in a us10 subaccount, while Automation Pilot remains in the ap21 subaccount.

## How to obtain support

Support for the content in this repository is available during the actual time of the online session for which this content has been designed. Otherwise, you may request support by sending an email request to the lectors at: biser.simeonov@sap.com or dimitar.donchev@sap.com

## License
Copyright (c) 2022 SAP SE or an SAP affiliate company. All rights reserved. This project is licensed under the Apache Software License, version 2.0 except as noted otherwise in the [LICENSE](LICENSES/Apache-2.0.txt) file.
