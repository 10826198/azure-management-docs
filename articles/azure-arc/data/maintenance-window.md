---
title: Maintenance Windows
description: Article describes how to set a maintenance window
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: grrlgeek
ms.author: jeschult
ms.reviewer: mikeray
ms.date: 02/17/2022
ms.topic: how-to
---

# Maintenance Window

A maintenance window can be configured on a data controller to define a time period in which upgrades will take place. In this time period, the Arc-enabled SQL Managed Instances on that data controller which have the "desiredVersion" property set to "auto" will be upgraded.

During setup, a duration, recurrence, and start date and time are specified. After the maintenance window starts, it will run for the period of time set in the duration. The instances attached to the data controller will begin upgrades (in parallel). At the end of the set duration, any upgrades that are in progress will continue to completion. Any instances that did not begin upgrading in the window will begin upgrading in the following recurrence.

## Prerequisites

An Azure Arc-enabled SQL Managed Instance with the ["desiredVersion" property set to auto](https://docs.microsoft.com/en-us/azure/azure-arc/data/upgrade-sql-managed-instance-auto).

## Limitations

The maintenance window duration can be from 2 hours to 8 hours.

Only one maintenance window can be set per data controller.

## Configure a maintenance window

The maintenance window has these settings:
- Duration - the length of time the window will run, expressed in hours and minutes (HH:MM).
- Recurrence - how often the window will occur. All words are case sensitive and must be capitalized. You can set weekly or monthly windows.
    - Weekly
        - [Week | Weekly][day of week]
        - Examples:
            - --recurrence  "Week Thursday"
            - --recurrence "Weekly Saturday"
	- Monthly
		- [Month | Monthly] [First | Second | Third | Fourth | Last] [day of week]
		- Examples:
			- --recurrence "Month Fourth Saturday"
			- --recurrence "Monthly Last Monday"
	- If recurrence is not specified, it will be a one-time maintenance window.
- Start - the date and time the first window will occur, in the format YYYY-MM-ddTHH:mm (24-hour format).
	- Example:
		- --start "2022-02-01T12:35"
- Time Zone - the [time zone](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones) associated with the maintenance window.

#### CLI

To create a maintenance window, use the following command:

```cli
az arcdata dc update mw --start <date and time> --duration <time> --recurrence <interval> --time-zone <time zone> --use-k8s 
```

Example:

```cli
az arcdata dc update mw --start "2022-01-01T23:00" --duration 3:00 --recurrence "Monthly First Saturday" --time-zone US/Pacific --use-k8s
```

## Monitor the upgrades

During the maintenance window, you can view the status of upgrades.

```kubectl
kubectl -n <namespace> get sqlmi -o yaml 
```

The ```status.runningVersion``` and ```status.lastUpdateTime``` fields will show the latest version and upgrade time.

## View existing maintenance window

You can view the maintenance window in the datacontroller spec. 

```kubectl
kubectl describe datacontroller -n <namespace>
```

Output:

```yaml
spec:
  settings:
    maintenance:
      duration: "3:00"
      recurrence: Monthly First Saturday
      start: "2022-01-01T23:00"
      timeZone: "US/Pacific"
```

## Failed upgrades

There is no automatic rollback for failed upgrades. If an instance failed to upgrade automatically, manual intervention will be needed to pin the instance to its current running version, using ```az sql mi-arc update```. 

```cli
az sql mi-arc update --name <instance name> --desired-version <version> 
```

Example:
```cli
az sql mi-arc update --name sql01 --desired-version v1.2.0_2021-12-15
```
