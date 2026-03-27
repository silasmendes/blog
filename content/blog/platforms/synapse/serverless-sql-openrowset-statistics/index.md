+++
date = '2026-03-27T00:00:00Z'
draft = true
title = 'Understanding OPENROWSET statistics in Synapse Serverless SQL pool'
+++

In the [previous post]({{< ref "blog/platforms/synapse/serverless-sql-external-tables-statistics/index.md" >}}), I discussed statistics for external tables. Today, let’s shift the focus to **statistics for OPENROWSET queries**.

If you are not familiar with OPENROWSET, it allows you to query data directly from a data lake without creating external tables. You simply write a SELECT statement that points to the file path, like this:

```sql
SELECT
    TOP 10 cc_call_center_id, cc_call_center_sk, cc_city
FROM
    OPENROWSET(
        BULK 'https://adls_path.dfs.core.windows.net/tables/tpcds_10gb/call_center/**',
        FORMAT = 'PARQUET'
    ) AS [result]
WHERE cc_call_center_id = 100
ORDER BY cc_city ASC
```

Just like with external tables, Serverless SQL pool relies on statistics to optimize OPENROWSET queries. These statistics describe how data is distributed across the files in the data lake, which helps the query optimizer estimate row counts, choose execution strategies, and allocate resources efficiently.

One major advantage compared to external tables is that Serverless automatically creates and updates statistics for OPENROWSET queries (there are other benefits for OPENROWSET, but we can cover those in a separate post). However, automatic behavior is not always enough.

In some scenarios, you may want tighter control over when statistics are created. Also, by default, Serverless typically builds statistics using only a small sample of the data. For most workloads this works well, but in certain edge cases (especially with highly skewed data), a small sample may not be accurate enough.

In those cases, creating statistics using `FULLSCAN` can improve query performance and stability.

### Creating OPENROWSET statistics manually

To manually create statistics for an OPENROWSET query, you must use the system stored procedure `sys.sp_create_openrowset_statistics`. This procedure expects a SELECT statement that reads only the column for which you want to build statistics.

For example, suppose you need to create stats for the column `cc_call_center_id` you need to write a simplified SELECT that:

* Reads only column `cc_call_center_id`
* Removes any filters
* Points to the exact same data source

Like this:

```sql
SELECT 
    cc_call_center_id
FROM
    OPENROWSET(
        BULK 'https://adls_path.dfs.core.windows.net/tables/tpcds_10gb/call_center/**',
        FORMAT = 'PARQUET'
    ) AS [result]
```

You then pass that SELECT statement as a parameter to the stored procedure (pay close attention to quoting):

```sql
EXECUTE sys.sp_create_openrowset_statistics @stmt = N'
SELECT 
    cc_call_center_id
FROM
    OPENROWSET(
        BULK ''https://adls_path.dfs.core.windows.net/tables/tpcds_10gb/call_center/**'',
        FORMAT = ''PARQUET''
    ) AS [result]
'    
```

Statistics created this way are built using FULLSCAN by default, so you do not need to specify it explicitly.

### Important characteristics and limitations

There are a few important behaviors to be aware of when working with OPENROWSET statistics.

- **Server-level scope**

OPENROWSET statistics are stored at the server level, not at the database level. If you create statistics while connected to database **X**, they will also be available to queries executed from database **Y** within the same Serverless SQL pool.

- **Single-column only**

Multi-column statistics are not supported. The SELECT used to create statistics must reference exactly one column. If multiple columns are included, the procedure will fail with an error similar to the one below:

```
Msg 15835, Level 16, State 1, Procedure sys.sp_create_openrowset_statistics
The query must project exactly one OPENROWSET column.
```

- **No visibility in system views**

Unlike statistics on external tables, OPENROWSET statistics are not exposed in `sys.stats`. Currently, there is no way to list existing OPENROWSET statistics or check when they were created.

- **No update operation**

OPENROWSET statistics cannot be updated. If you need fresher statistics, the only option is to drop and recreate them.

- **Cleaning up statistics**

If needed, you can also **remove all OPENROWSET statistics** from your Serverless SQL pool using the command below:

```sql
EXECUTE sp_cleanup_all_openrowset_statistics
```

>⚠️ **Warning:** Removing all statistics at once can have performance implications. When users subsequently run queries that read the underlying data, Serverless SQL pool will automatically create statistics. This can trigger multiple auto-create statistics operations simultaneously, potentially increasing query latency and resource consumption for the first workloads that access the data.

### A helper script for managing OPENROWSET statistics

To make it easier to create statistics for many columns, I built a Python script that simplifies this process: [synapse_create_drop_openrowset_stats.py](https://github.com/silasmendes/python-library/blob/main/synapse_create_drop_openrowset_stats.py).

See you in the next post :)