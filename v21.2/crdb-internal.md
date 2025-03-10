---
title: crdb_internal
summary: The crdb_internal schema contains read-only views that you can use for introspection into CockroachDB internals.
toc: true
---

The `crdb_internal` [system catalog](system-catalogs.html) is a schema that contains information about internal objects, processes, and metrics related to a specific database.

{{site.data.alerts.callout_danger}}
We do not recommend using `crdb_internal` tables in production environments for the following reasons:

- The contents of `crdb_internal` are unstable, and subject to change in new releases of CockroachDB, without prior notice.
- There are memory and latency costs associated with each table in `crdb_internal`. Accessing the tables in the schema can impact cluster stability and performance.
{{site.data.alerts.end}}

## Data exposed by `crdb_internal`

Each table in `crdb_internal` corresponds to an internal object, process, or metric, for a specific database. `crdb_internal` tables are read-only.

In CockroachDB {{ page.version.version }}, `crdb_internal` includes the following tables:

Table | Description
-------|-------
`active_range_feeds` | Contains information about [range feeds](architecture/distribution-layer.html) on nodes in your cluster.
`backward_dependencies` | Contains information about backward dependencies.
`builtin_functions` | Contains information about supported [functions](functions-and-operators.html).
`cluster_contended_indexes` | Contains information about [contended](performance-best-practices-overview.html#understanding-and-avoiding-transaction-contention) indexes in your cluster.
`cluster_contended_keys` | Contains information about [contended](performance-best-practices-overview.html#understanding-and-avoiding-transaction-contention) keys in your cluster.
`cluster_contended_tables` | Contains information about [contended](performance-best-practices-overview.html#understanding-and-avoiding-transaction-contention) tables in your cluster.
`cluster_contention_events` | Contains information about [contention](performance-best-practices-overview.html#understanding-and-avoiding-transaction-contention) in your cluster.
`cluster_database_privileges` | Contains information about the [database privileges](authorization.html#privileges) on your cluster.
`cluster_distsql_flows` | Contains information about the flows of the [DistSQL execution](architecture/sql-layer.html#distsql) scheduled in your cluster.
`cluster_inflight_traces` | Contains information about in-flight [tracing](show-trace.html) in your cluster.
`cluster_queries` | Contains information about queries running on your cluster.
`cluster_sessions` | Contains information about cluster sessions, including current and past queries.
`cluster_settings` | Contains information about [cluster settings](cluster-settings.html).
`cluster_transactions` | Contains information about transactions running on your cluster.
`create_statements` | Contains information about tables and indexes in your database.
`create_type_statements` | <a name="create_type_statements"></a> Contains information about [user-defined types](enum.html) in your database.
`cross_db_references` | Contains information about objects that reference other objects, such as [foreign keys](foreign-key.html) or [views](views.html), across databases in your cluster.
`databases` | Contains information about the databases in your cluster.
`default_privileges` | Contains information about per-database default [privileges](authorization.html#privileges).
`feature_usage` | Contains information about feature usage on your cluster.
`forward_dependencies` | Contains information about forward dependencies.
`gossip_alerts` | Contains information about gossip alerts.
`gossip_liveness` | Contains information about your cluster's gossip liveness.
`gossip_network` | Contains information about your cluster's gossip network.
`gossip_nodes` | Contains information about nodes in your cluster's gossip network.
`index_columns` | Contains information about [indexed](indexes.html) columns in your cluster.
`index_usage_statistics` | Contains statistics about the primary and secondary indexes used in statements.
`interleaved` | Contains information about [interleaved objects](interleave-in-parent.html) in your cluster.
`invalid_objects` | Contains information about invalid objects in your cluster.
`jobs` | Contains information about [jobs](show-jobs.html) running on your cluster.
`kv_node_liveness` | Contains information about [node liveness](cluster-setup-troubleshooting.html#node-liveness-issues).
`kv_node_status` | Contains information about node status at the [key-value layer](architecture/storage-layer.html).
`kv_store_status` | Contains information about the key-value store for your cluster.
`leases` | Contains information about [leases](architecture/replication-layer.html#leases) in your cluster.
`lost_descriptors_with_data` | Contains information about table descriptors that have been deleted but still have data left over in storage.
`node_build_info` | Contains information about nodes in your cluster.
`node_contention_events` | Contains information about contention on the gateway node of your cluster.
`node_distsql_flows` | Contains information about the flows of the [DistSQL execution](architecture/sql-layer.html#distsql) scheduled on nodes in your cluster.
`node_inflight_trace_spans` | Contains information about currently in-flight spans in the current node.
`node_metrics` | Contains metrics for nodes in your cluster.
`node_queries` | Contains information about queries running on nodes in your cluster.
`node_runtime_info` | Contains runtime information about nodes in your cluster.
`node_sessions` | Contains information about sessions to nodes in your cluster.
`node_statement_statistics` | Contains statement statistics for nodes in your cluster.
`node_transaction_statistics` | Contains transaction statistics for nodes in your cluster.
`node_transactions` | Contains information about transactions for nodes in your cluster.
`node_txn_stats` | Contains transaction statistics for nodes in your cluster.
`partitions` | Contains information about [partitions](partitioning.html) in your cluster.
`predefined_comments` | Contains predefined comments about your cluster.
`ranges` | Contains information about ranges in your cluster.
`ranges_no_leases` | Contains information about ranges in your cluster, without leases.
`regions` | Contains information about [cluster regions](multiregion-overview.html#cluster-regions).
`schema_changes` | Contains information about schema changes in your cluster.
`session_trace` | Contains session trace information for your cluster.
`session_variables` | Contains information about [session variables](set-vars.html) in your cluster.
`statement_statistics` | Table that aggregates in-memory and persisted [statistics](ui-statements-page.html) from `system.statement_statistics` within hourly time intervals based on UTC time, rounded down to the nearest hour. You can manually reset the statistics by calling `SELECT crdb_internal.reset_sql_stats()`.
`table_columns` | Contains information about table columns in your cluster.
`table_indexes` | Contains information about table indexes in your cluster.
`table_row_statistics` | Contains row count statistics for tables in the current database.
`tables` | Contains information about tables in your cluster.
`transaction_statistics` | Table that aggregates in-memory and persisted [statistics](ui-transactions-page.html) from `system.transaction_statistics` within hourly time intervals based on UTC time, rounded down to the nearest hour. You can manually reset the statistics by calling `SELECT crdb_internal.reset_sql_stats()`.
`zones` | Contains information about [zone configurations](configure-replication-zones.html) in your cluster.

To list the `crdb_internal` tables for the [current database](sql-name-resolution.html#current-database), use the following [`SHOW TABLES`](show-tables.html) statement:

{% include copy-clipboard.html %}
~~~ sql
> SHOW TABLES FROM crdb_internal;
~~~

~~~
   schema_name  |         table_name          | type  | owner | estimated_row_count
----------------+-----------------------------+-------+-------+----------------------
  crdb_internal | backward_dependencies       | table | NULL  |                NULL
  crdb_internal | builtin_functions           | table | NULL  |                NULL
  ...
~~~

## Querying `crdb_internal` tables

To get detailed information about objects, processes, or metrics related to your database, you can read from the `crdb_internal` table that corresponds to the item of interest.

{{site.data.alerts.callout_success}}
To ensure that you can view all of the tables in `crdb_internal`, query the tables as a user with [`admin` privileges](authorization.html#admin-role).
{{site.data.alerts.end}}

{{site.data.alerts.callout_info}}
Unless specified otherwise, queries to `crdb_internal` assume the [current database](sql-name-resolution.html#current-database).
{{site.data.alerts.end}}

For example, to return the `crdb_internal` table for the ranges of the [`movr`](movr.html) database, you can use the following statement:

{% include copy-clipboard.html %}
~~~ sql
> SELECT * FROM movr.crdb_internal.ranges;
~~~

~~~
  range_id |                                                                         start_key                                                                          |                                        start_pretty                                         |                                                                          end_key                                                                           |                                         end_pretty                                          | database_name |           table_name            | index_name | replicas |    replica_localities    | learner_replicas |       split_enforced_until       | lease_holder | range_size
-----------+------------------------------------------------------------------------------------------------------------------------------------------------------------+---------------------------------------------------------------------------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------+---------------------------------------------------------------------------------------------+---------------+---------------------------------+------------+----------+--------------------------+------------------+----------------------------------+--------------+-------------
         1 |                                                                                                                                                            | /Min                                                                                        | \004\000liveness-                                                                                                                                          | /System/NodeLiveness                                                                        |               |                                 |            | {1}      | {"region=us-east1,az=b"} | {}               | NULL                             |            1 |       8323
         2 | \004\000liveness-                                                                                                                                          | /System/NodeLiveness                                                                        | \004\000liveness.                                                                                                                                          | /System/NodeLivenessMax                                                                     |               |                                 |            | {1}      | {"region=us-east1,az=b"} | {}               | NULL                             |            1 |       9278
         3 | \004\000liveness.                                                                                                                                          | /System/NodeLivenessMax                                                                     | \004tsd                                                                                                                                                    | /System/tsd                                                                                 |               |                                 |            | {1}      | {"region=us-east1,az=b"} | {}               | NULL                             |            1 |      32377
         4 | \004tsd                                                                                                                                                    | /System/tsd                                                                                 | \004tse                                                                                                                                                    | /System/"tse"                                                                               |               |                                 |            | {1}      | {"region=us-east1,az=b"} | {}               | NULL                             |            1 |    4067446
(65 rows)
~~~

## See also

- [`SHOW`](show-vars.html)
- [`SHOW COLUMNS`](show-columns.html)
- [`SHOW CONSTRAINTS`](show-constraints.html)
- [`SHOW CREATE`](show-create.html)
- [`SHOW DATABASES`](show-databases.html)
- [`SHOW GRANTS`](show-grants.html)
- [`SHOW INDEX`](show-index.html)
- [`SHOW SCHEMAS`](show-schemas.html)
- [`SHOW TABLES`](show-tables.html)
- [SQL Name Resolution](sql-name-resolution.html)
- [System Catalogs](system-catalogs.html)
