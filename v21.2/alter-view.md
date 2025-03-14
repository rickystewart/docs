---
title: ALTER VIEW
summary: The ALTER VIEW statement applies a schema change to a view.
toc: true
---

The `ALTER VIEW` [statement](sql-statements.html) applies a schema change to a [view](views.html).

{% include {{{ page.version.version }}/misc/schema-change-stmt-note.md %}

## Required privileges

- To alter a view, the user must have the `CREATE` [privilege](authorization.html#assign-privileges) on the parent database.
- To change the schema of a view with `ALTER VIEW ... SET SCHEMA`, or to change the database of a view with `ALTER VIEW ... RENAME TO`, the user must also have the `DROP` [privilege](authorization.html#assign-privileges) on the view.

## Syntax

<div>
{% remote_include https://raw.githubusercontent.com/cockroachdb/generated-diagrams/release-21.2/grammar_svg/alter_view.html %}
</div>

## Parameters

Parameter | Description
----------|------------
`MATERIALIZED` |  Rename a [materialized view](views.html#materialized-views).
`IF EXISTS` | Rename the view only if a view of `view_name` exists; if one does not exist, do not return an error.
`view_name` | The name of the view to rename. To find view names, use:<br><br>`SELECT * FROM information_schema.tables WHERE table_type = 'VIEW';`
`RENAME TO view_name` | Rename the view to `view_name`, which must be unique to its database and follow these [identifier rules](keywords-and-identifiers.html#identifiers). Name changes do not propagate to the  table(s) using the view.<br><br>Note that `RENAME TO` can be used to move a view from one database to another, but it cannot be used to move a view from one schema to another. To change a view's schema, use `ALTER VIEW ...SET SCHEMA` instead. In a future release, `RENAME TO` will be limited to changing the name of a view, and will not have the ability to change a view's database.
`SET SCHEMA schema_name` | Change the schema of the view to `schema_name`.
`OWNER TO role_spec` |  Change the owner of the view to `role_spec`.

## Limitations

CockroachDB does not currently support:

- Changing the [`SELECT`](select-clause.html) statement executed by a view. Instead, you must drop the existing view and create a new view.
- Renaming a view that other views depend on. This feature may be added in the future (see [tracking issue](https://github.com/cockroachdb/cockroach/issues/10083)).

## Examples

### Rename a view

Suppose you create a new view that you want to rename:

{% include copy-clipboard.html %}
~~~ sql
> CREATE VIEW money_rides (id, revenue) AS SELECT id, revenue FROM rides WHERE revenue > 50;
~~~

{% include copy-clipboard.html %}
~~~ sql
> SELECT * FROM [SHOW TABLES] WHERE type = 'view';
~~~

~~~
  schema_name | table_name  | type | owner | estimated_row_count | locality
--------------+-------------+------+-------+---------------------+-----------
  public      | money_rides | view | demo  |                   0 | NULL
(1 row)
~~~

{% include copy-clipboard.html %}
~~~ sql
> ALTER VIEW money_rides RENAME TO expensive_rides;
~~~
~~~
RENAME VIEW
~~~

{% include copy-clipboard.html %}
~~~ sql
> SELECT * FROM [SHOW TABLES] WHERE type = 'view';
~~~

~~~
  schema_name |   table_name    | type | owner | estimated_row_count | locality
--------------+-----------------+------+-------+---------------------+-----------
  public      | expensive_rides | view | demo  |                   0 | NULL
(1 row)
~~~

### Change the schema of a view

Suppose you want to add the `expensive_rides` view to a schema called `cockroach_labs`:

By default, [unqualified views](sql-name-resolution.html#lookup-with-unqualified-names) created in the database belong to the `public` schema:

{% include copy-clipboard.html %}
~~~ sql
> SHOW CREATE public.expensive_rides;
~~~

~~~
        table_name       |                                                 create_statement
-------------------------+-------------------------------------------------------------------------------------------------------------------
  public.expensive_rides | CREATE VIEW public.expensive_rides (id, revenue) AS SELECT id, revenue FROM movr.public.rides WHERE revenue > 50
(1 row)
~~~

If the new schema does not already exist, [create it](create-schema.html):

{% include copy-clipboard.html %}
~~~ sql
> CREATE SCHEMA IF NOT EXISTS cockroach_labs;
~~~

Then, change the view's schema:

{% include copy-clipboard.html %}
~~~ sql
> ALTER VIEW expensive_rides SET SCHEMA cockroach_labs;
~~~

{% include copy-clipboard.html %}
~~~ sql
> SHOW CREATE public.expensive_rides;
~~~

~~~
ERROR: relation "public.expensive_rides" does not exist
SQLSTATE: 42P01
~~~

{% include copy-clipboard.html %}
~~~ sql
> SHOW CREATE cockroach_labs.expensive_rides;
~~~

~~~
            table_name           |                                                     create_statement
---------------------------------+---------------------------------------------------------------------------------------------------------------------------
  cockroach_labs.expensive_rides | CREATE VIEW cockroach_labs.expensive_rides (id, revenue) AS SELECT id, revenue FROM movr.public.rides WHERE revenue > 50
(1 row)
~~~

## See also

- [Views](views.html)
- [`CREATE VIEW`](create-view.html)
- [`SHOW CREATE`](show-create.html)
- [`DROP VIEW`](drop-view.html)
- [Online Schema Changes](online-schema-changes.html)
