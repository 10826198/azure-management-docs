--- 
title: Get connection endpoints and form connection strings for your Arc enabled PostgreSQL Hyperscale server group
titleSuffix: Azure Arc enabled data services
description: Get connection endpoints and form connection strings for your Arc enabled PostgreSQL Hyperscale server group
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: TheJY
ms.author: jeanyd
ms.reviewer: mikeray
ms.date: 08/04/2020
ms.topic: how-to
---

# Get connection endpoints and form connection strings for your Arc enabled PostgreSQL Hyperscale server group

This article explains how you can retrieve the connection endpoints for your server group and how you form connection strings you will use with your applications and/or tools.


## Get connection end points:

### From CLI with azdata
#### 1. Connect to your Arc Data Controller:
- If you already have a session opened on the host of the Arc Data Controller:
Run the following command:
```terminal
azdata login
```

- If you do not have a session opened on the host of the Arc Data Controller:
run the following command 
```terminal
azdata login --endpoint https://<external IP address of host/data controller>:30080
```

#### 2. Show the connection endpoints
Run the following command:
```terminal
azdata arc postgres server endpoint list -n <server group name>
```
It returns an output like:
```terminal
Description           Endpoint
--------------------  ------------------------------------------------------------------------------------------------------------------------
PostgreSQL Instance   postgresql://postgres:<replace with password>@<IPaddress>:<port number>
Log Search Dashboard  https://<IPaddress>:<port number>/kibana/app/kibana#/discover?_a=(query:(language:kuery,query:'custom_resource_name:"<server group name>"'))
Metrics Dashboard     https://<IPaddress>:<port number>/grafana/d/postgres-metrics?var-Namespace=arc&var-Name=<server group name>
```
Use these end points to:
- Form your connection strings and connect with your client tools or applications
- Access the Grafana and Kibana dashboards from your browser

For example, you can use the end point named _PostgreSQL Instance_ to connect with psql to your server group. For example:
```terminal
psql postgresql://postgres:MyPassworkd@123.456.789.111:31066
psql (10.14 (Ubuntu 10.14-0ubuntu0.18.04.1), server 12.4 (Ubuntu 12.4-1.pgdg16.04+1))
WARNING: psql major version 10, server major version 12.
         Some psql features might not work.
SSL connection (protocol: TLSv1.2, cipher: ECDHE-RSA-AES256-GCM-SHA384, bits: 256, compression: off)
Type "help" for help.

postgres=#
```
> [!NOTE]
>
> - The password of the _postgresql_ user indicated in the end point named "_PostgreSQL Instance_ is the password you chose when deploying the server group.
> - About azdata: the lease associated to your connection lasts about 10 hours. After that you need to reconnect. If your lease has expired, you will get the following error message when you try to execute a command with azdata (other than azdata login):
> _ERROR: (401)_
> _Reason: Unauthorized_
> _HTTP response headers: HTTPHeaderDict({'Date': 'Sun, 06 Sep 2020 16:58:38 GMT', 'Content-Length': '0', 'WWW-Authenticate': '_
> _Basic realm="Login_ credentials required", Bearer error="invalid_token", error_description="The token is expired"'})_
> When this happens, you need to reconnect with azdata as explained above.

## From CLI with kubectl
- If your server group is of Postgres version 12 (default), then the following command:
```terminal
kubectl get postgresql-12/<server group name>
```
- If your server group is of Postgres version 11, then the following command:
```terminal
kubectl get postgresql-11/<server group name>
```

Those commands will produce output like the one below. You can use that information to form your connection strings:
```terminal
NAME         STATE   READY-PODS   EXTERNAL-ENDPOINT   AGE
postgres01   Ready   3/3          123.456.789.4:31066      5d20h
```` 


## Form connection strings:
Use the below table of templates of connections strings for your server group. You can then copy/paste and customize them as further needed:

:::row:::
   :::column span="":::
**ADO.NET**
   :::column-end:::
   :::column span="":::
`Server=192.168.1.121;Database=postgres;Port=24276;User Id=postgres;Password={your_password_here};Ssl Mode=Require;`
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
**C++ (libpq)**
   :::column-end:::
   :::column span="":::
host=192.168.1.121 port=24276 dbname=postgres user=postgres password={your_password_here} sslmode=require
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
**JDBC**
   :::column-end:::
   :::column span="":::
jdbc:postgresql://192.168.1.121:24276/postgres?user=postgres&password={your_password_here}&sslmode=require
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
**Node.js**
   :::column-end:::
   :::column span="":::
host=192.168.1.121 port=24276 dbname=postgres user=postgres password={your_password_here} sslmode=require
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
**PHP**
   :::column-end:::
   :::column span="":::
host=192.168.1.121 port=24276 dbname=postgres user=postgres password={your_password_here} sslmode=require
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
**psql**
   :::column-end:::
   :::column span="":::
psql "host=192.168.1.121 port=24276 dbname=postgres user=postgres password={your_password_here} sslmode=require"
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
**Python**
   :::column-end:::
   :::column span="":::
dbname='postgres' user='postgres' host='192.168.1.121' password='{your_password_here}' port='24276' sslmode='true'
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
**Ruby**
   :::column-end:::
   :::column span="":::
host=192.168.1.121; dbname=postgres user=postgres password={your_password_here} port=24276 sslmode=require
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
**Web App**
   :::column-end:::
   :::column span="":::
Database=postgres; Data Source=192.168.1.121; User Id=postgres; Password={your_password_here}
   :::column-end:::
:::row-end:::


## Next steps
- Read about [scaling out (adding worker nodes)](scale-out-postgresql-hyperscale-server-group.md) your server group
- Read about [scaling up or down (increasing/decreasing memory/vcores)](scale-up-down-postgresql-hyperscale-server-group-using-cli.md) your server group


