---
title: Scale out your Azure Database for PostgreSQL Hyperscale server group
description: Scale out your Azure Database for PostgreSQL Hyperscale server group
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: twright-msft
ms.author: twright
ms.reviewer: mikeray
ms.date: 08/04/2020
ms.topic: how-to
---

# Scenario: Scale out your Azure Database for PostgreSQL Hyperscale server group

These instructions utilize the PostgreSQL server group that was [provisioned in an earlier guide](https://github.com/microsoft/Azure-data-services-on-Azure-Arc/blob/ctp2.0/scenarios/004-create-pghsaa-instance.md).

>**NOTE:** It is not yet possible to scale back in, i.e. reduce the number of worker nodes. If you need to do so, you need to extract the data, drop the server group, create a new server group with less worker nodes and then import the data.

## Load test data

First of all, to test scaling out, its helpful to have test data. We utilize a sample of publicly available GitHub data, available from the Citus Data website (Citus Data is part of Microsoft).

Let's connect to our database by first getting the connection information:

```terminal
azdata arc postgres server endpoint list -n <server name> -ns <namespace name>

#Example:
#azdata arc postgres server endpoint list -n postgres01 -ns arc
```

Example output:

```terminal
Description           Endpoint
--------------------  ----------------------------------------------------------------------------------------------------------------------------
PostgreSQL Instance   postgresql://postgres:<replace with password>@10.240.0.6:31787
Log Search Dashboard  https://52.152.248.25:30777/kibana/app/kibana#/discover?_a=(query:(language:kuery,query:'kubernetes_pod_name:"postgres01"'))
Metrics Dashboard     https://52.152.248.25:30777/grafana/d/postgres-metrics?var-Namespace=arc&var-Name=postgres01
```

Now, connect to the Postgres instance using Azure Data Studio.

Open a new query window abd run the following query to verify that we currently have two or more Hyperscale worker nodes, each corresponding to a Kubernetes pod:

```sql
SELECT * FROM pg_dist_node;
```

```terminal
 nodeid | groupid |                       nodename                        | nodeport | noderack | hasmetadata | isactive | noderole | nodecluster | metadatasynced | shouldhaveshards
--------+---------+-------------------------------------------------------+----------+----------+-------------+----------+----------+-------------+----------------+------------------
      1 |       1 | pg1-1.pg1-svc.default.svc.cluster.local |     5432 | default  | f           | t        | primary  | default     | f              | t
      2 |       2 | pg1-2.pg1-svc.default.svc.cluster.local |     5432 | default  | f           | t        | primary  | default     | f              | t
(2 rows)
```

Now we’re going to set up our two tables by running the following query:

```sql
CREATE TABLE github_events
(
    event_id bigint,
    event_type text,
    event_public boolean,
    repo_id bigint,
    payload jsonb,
    repo jsonb,
    user_id bigint,
    org jsonb,
    created_at timestamp
);

CREATE TABLE github_users
(
    user_id bigint,
    url text,
    login text,
    avatar_url text,
    gravatar_id text,
    display_login text
);
```

On the payload field of events we have a JSONB datatype. JSONB is the JSON datatype in binary form in Postgres. This makes it easy to store a more flexible schema in a single column and with Postgres we can create a GIN index on this which will index every key and value within it. With a GIN index it becomes fast and easy to query with various conditions directly on that payload. So we’ll go ahead and create a couple of indexes before we load our data:

```sql
CREATE INDEX event_type_index ON github_events (event_type);
CREATE INDEX payload_index ON github_events USING GIN (payload jsonb_path_ops);
```

Next we’ll actually take those standard Postgres tables and tell Postgres to shard them out. To do so we’ll run a query for each table. With this query we’ll specify the table we want to shard, as well as the key we want to shard it on. In this case we’ll shard both the events and users table on user_id:

```sql
SELECT create_distributed_table('github_events', 'user_id');
SELECT create_distributed_table('github_users', 'user_id');
```

Now, Load the data with COPY ... FROM PROGRAM:

```sql
COPY github_users FROM PROGRAM 'curl "https://examples.citusdata.com/users.csv"' WITH ( FORMAT CSV );
COPY github_events FROM PROGRAM 'curl "https://examples.citusdata.com/events.csv"' WITH ( FORMAT CSV );
```

And now lets take a measurement for how long a simple query takes with two nodes:

```sql
SELECT COUNT(*) FROM github_events;
```

Look at the query execution duration by clicking on the Messages tab.

```terminal
Started executing query at Line 1
(1 row(s) affected)
Total execution time: 00:00:00.442
```

## Scaling out

Increase the number of worker nodes to 4, by running the following command:

```terminal
azdata arc postgres server edit -n <name of your postgresql server group> -ns <name of the namespace> -w <number of workers>

#Example:
#azdata arc postgres server edit -n postgres01 -ns arc -w 4
```

This will first start adding the nodes, and you'll see a Pending state for the server group:

```terminal
azdata arc postgres server list
```

```terminal
ClusterIP         ExternalIP      MustRestart    Name        Status
----------------  --------------  -------------  ----------  -----------
10.98.62.6:31815  10.0.0.4:31815  False          postgres01  Pending 4/5
```

Once the nodes are available, the Hyperscale Shard Rebalancer runs automatically, and redistributes the data to the new nodes.

> [!NOTE]
>  During this scale out operation the database remains fully online, and we can continue running queries.

You can verify that the new nodes are available:

```sql
SELECT * FROM pg_dist_node;
```

```terminal
 nodeid | groupid |                       nodename                        | nodeport | noderack | hasmetadata | isactive | noderole | nodecluster | metadatasynced | shouldhaveshards
--------+---------+-------------------------------------------------------+----------+----------+-------------+----------+----------+-------------+----------------+------------------
      1 |       1 | pg1-1.pg1-svc.default.svc.cluster.local |     5432 | default  | f           | t        | primary  | default     | f              | t
      2 |       2 | pg1-2.pg1-svc.default.svc.cluster.local |     5432 | default  | f           | t        | primary  | default     | f              | t
      3 |       3 | pg1-3.pg1-svc.default.svc.cluster.local |     5432 | default  | f           | t        | primary  | default     | f              | t
      4 |       4 | pg1-4.pg1-svc.default.svc.cluster.local |     5432 | default  | f           | t        | primary  | default     | f              | t
(4 rows)
```

And now the same count query can be performed across the four worker nodes, without any changes in the SQL statement.

If you deploy on a single VM for testing this likely has similar runtime as before, if you deploy on a production-sized multi-node cluster, the performance should have improved:

```sql
SELECT COUNT(*) FROM github_events;
```

```terminal
Started executing query at Line 1
(1 row(s) affected)
Total execution time: 00:00:00.362
```

## Next Steps

Try out [Backup and Restore for Azure Database for PostgreSQL Hyperscale server groups](backup-restore-pghsaa.md).
