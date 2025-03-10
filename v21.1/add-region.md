---
title: ADD REGION
summary: The ADD REGION statement adds a region to a multi-region database.
toc: true
---

<span class="version-tag">New in v21.1:</span> The `ALTER DATABASE .. ADD REGION` [statement](sql-statements.html) adds a [region](multiregion-overview.html#database-regions) to a [multi-region database](multiregion-overview.html).

{% include enterprise-feature.md %}

{{site.data.alerts.callout_info}}
`ADD REGION` is a subcommand of [`ALTER DATABASE`](alter-database.html).
{{site.data.alerts.end}}

{{site.data.alerts.callout_danger}}
In order to add a region with `ADD REGION`, you must first set a primary database region with [`SET PRIMARY REGION`](set-primary-region.html), or at [database creation](create-database.html).<br>For an example showing how to add a primary region with `ALTER DATABASE`, see [Set the primary region](#set-the-primary-region).
{{site.data.alerts.end}}

## Synopsis

<div>
{% include {{ page.version.version }}/sql/generated/diagrams/alter_database_add_region.html %}
</div>

## Parameters

| Parameter       | Description                                                                                                                                                       |
|-----------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `database_name` | The database to which you are adding a [region](multiregion-overview.html#database-regions).                                                                      |
| `region_name`   | The [region](multiregion-overview.html#database-regions) being added to this database.  Allowed values include any region present in `SHOW REGIONS FROM CLUSTER`. |

## Required privileges

To add a region to a database, the user must have one of the following:

- Membership to the [`admin`](authorization.html#roles) role for the cluster.
- Membership to the [owner](authorization.html#object-ownership) role, or the [`CREATE` privilege](authorization.html#supported-privileges), for the database and all [`REGIONAL BY ROW`](multiregion-overview.html#regional-by-row-tables) tables in the database.

## Examples

{% include {{page.version.version}}/sql/multiregion-example-setup.md %}

### Set the primary region

Suppose you have a database `foo` in your cluster, and you want to make it a multi-region database.

To add the first region to the database, or to set an already-added region as the primary region, use a [`SET PRIMARY REGION`](set-primary-region.html) statement:

{% include copy-clipboard.html %}
~~~ sql
ALTER DATABASE foo SET PRIMARY REGION "us-east1";
~~~

~~~
ALTER DATABASE PRIMARY REGION
~~~

Given a cluster with multiple regions, any databases in that cluster that have not yet had their primary regions set will have their replicas spread as broadly as possible for resiliency. When a primary region is added to one of these databases:

- All tables will be [`REGIONAL BY TABLE`](set-locality.html#regional-by-table) in the primary region by default.
- This means that all such tables will have all of their voting replicas and leaseholders moved to the primary region. This process is known as [rebalancing](architecture/replication-layer.html#leaseholder-rebalancing).

### Add regions to a database

To add more regions to a database that already has at least one region, use an `ADD REGION` statement:

{% include copy-clipboard.html %}
~~~ sql
ALTER database foo ADD region "us-west1";
~~~

~~~
ALTER DATABASE ADD REGION
~~~

{% include copy-clipboard.html %}
~~~ sql
ALTER database foo ADD region "europe-west1";
~~~

~~~
ALTER DATABASE ADD REGION
~~~

### View a database's regions

To view the regions associated with a multi-region database, use a [`SHOW REGIONS FROM DATABASE`](show-regions.html) statement:

{% include copy-clipboard.html %}
~~~ sql
SHOW REGIONS FROM DATABASE foo;
~~~

~~~
  database |    region    | primary |  zones
-----------+--------------+---------+----------
  foo      | us-east1     |  true   | {b,c,d}
  foo      | europe-west1 |  false  | {b,c,d}
  foo      | us-west1     |  false  | {a,b,c}
(3 rows)
~~~

### Drop a region from a database

To [drop a region](drop-region.html) from a multi-region database, use a [`DROP REGION`](drop-region.html) statement:

{% include copy-clipboard.html %}
~~~ sql
ALTER DATABASE foo DROP REGION "us-west1";
~~~

~~~
ALTER DATABASE DROP REGION
~~~

{% include copy-clipboard.html %}
~~~ sql
SHOW REGIONS FROM DATABASE foo;
~~~

~~~
  database |  region  | primary |  zones
-----------+----------+---------+----------
  foo      | us-east1 |  true   | {b,c,d}
(1 row)
~~~

## See also

- [Multi-region overview](multiregion-overview.html)
- [`SET PRIMARY REGION`](set-primary-region.html)
- [`DROP REGION`](drop-region.html)
- [`SHOW REGIONS`](show-regions.html)
- [`ALTER TABLE`](alter-table.html)
- [Other SQL Statements](sql-statements.html)
