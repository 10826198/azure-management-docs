---
title: Introduction to Azure Arc-enabled data services with Active Directory authentication
description: Introduction to Azure Arc-enabled data services with Active Directory authentication
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: cloudmelon
ms.author: melqin
ms.reviewer: mikeray
ms.date: 04/05/2022
ms.topic: how-to
---

# Introduction to Azure Arc-enabled SQL Managed Instance with Active Directory authentication 

This article describes enabling Azure Arc-enabled SQL Managed Instance with Active Directory (AD) Authentication, with two possible Active Directory integration modes: 
-  Bring your own keytab (BYOK) mode 
-  Automatic mode 

Active Directory integration modes describes the management the keytab file

## Background

Arc-enabled data services support Active Directory (AD) for Identity and Access Management (IAM). The Arc-enabled SQL Managed instances uses an existing on-premises Active Directory (AD) domain for authentication. Users need to follow the following steps to enable Active Directory authentication for Arc-enabled SQL Managed Instance : 

- [Deploy data controller](create-data-controller-indirect-cli.md) 
- [Deploy a Bring your own keytab (BYOK) AD connector](deploy-byok-active-directory-connector.md) or [Deploy an automatic AD connector](deploy-automatic-active-directory-connector.md)
- [Deploy SQL Managed instances](deploy-active-directory-sql-managed-instance.md)

The following diagram shows how user proceed to enable Active Directory authentication for Arc-enabled SQL Managed Instance :

![Actice Directory Deployment User journey](media/active-directory-deployment/active-directory-user-journey.png)


## What is an Active Directory (AD) connector?

In order to enable Active Directory authentication for SQL Managed Instance, a SQL Managed Instance must be deployed in an environment that allows it to communicate with the Active Directory domain. 

To facilitate this, Azure Arc introduces a new Kubernetes-native [Custom Resource Definition (CRD)](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/) called `Active Directory Connector`, it provide arc-enabled SQL Managed Instances running on the same Data Controller an ability to perform Active Directory Authentication.


## What is the difference between a Bring your own keytab (BYOK) Active Directory (AD) connector and Automatic Active Directory (AD) connector ?

To enable Active Directory Authentication for Arc-enabled SQL Managed Instances, you need an Active Directory (AD) connector where you dermine the mode of the AD deployment : Bring your own keytab (BYOK) or Automatic. 

In the Bring your own keytab (BYOK) mode, users will bring in : 
- A a pre-created Active Directory (AD) account prior to the AD deployment
- Service Principal Names (SPNs) under that AD account
- Your own [Keytab file](/sql/linux/sql-server-linux-ad-auth-understanding#what-is-a-keytab-file)

When you deploy the Bring your own keytab (BYOK) AD connector, it is up to users to create the AD account, take care of the SPN registration and create the keytab file. You can create then using [Active Directory utility (adutil)](/sql/linux/sql-server-linux-ad-auth-adutil-introduction).

To know further about how to [deploy a Bring your own keytab (BYOK) Active Directory (AD) connector](deploy-automatic-active-directory-connector.md)

In the automatic mode, you need an automatic Active Directory (AD) connector, you will bring an Organisational Unit (OU) and an AD domain service account has sufficient permissions in the Active Directory. 

Furthermore,  the system will take care of the following work : 
- An domain service AD account is automatically generated for SQL managed instance.
- The system sets SPNs automatically on that AD account.
- A keytab file is generated then delivered to the SQL Managed instance.

The mode of the AD connector is determined by the value of **spec.activeDirectory.serviceAccountProvisioning** which can be set to Bring your own keytab (BYOK) or automatic. Note that once this parameter is set to automatic, the following parameter becomes mandatory too : 
- spec.activeDirectory.ouDistinguishedName
- spec.activeDirectory.domainServiceAccountSecret

When a SQL Managed Instance is deployed with the intention to enable Active Directory Authentication, it needs to reference the Active Directory Connector instance it wants to use. Referencing the Active Directory Connector in SQL MI spec will automatically set up the needed environment in the SQL Managed Instance container for SQL MI to perform Active Directory authentication. 

To know further about how to [Deploy an automatic Active Directory (AD) connector](deploy-automatic-active-directory-connector.md)

## Next steps

* [Deploy an Bring your own keytab (BYOK) Active Directory (AD) connector](deploy-byok-active-directory-connector.md)
* [Deploy an automatic Active Directory (AD) connector](deploy-automatic-active-directory-connector.md)
* [Deploy Azure Arc-enabled SQL Managed Instance in Active Directory (AD)](deploy-active-directory-sql-managed-instance.md)
* [Connect to AD-integrated Azure Arc-enabled SQL Managed Instance](connect-active-directory-sql-managed-instance.md)
