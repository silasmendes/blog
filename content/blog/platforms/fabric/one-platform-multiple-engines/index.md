+++
date = '2026-03-27T00:00:00Z'
draft = true
title = 'Microsoft Fabric - One plataform, multiple engines'
+++

Modern data analytics solutions such as **Azure Synapse Analytics**,  **Microsoft Fabric** and **Databricks** are not single products. They are collections of different engines and services working together. What they do extremely well is hide that complexity through strong integration.

Take **Microsoft Fabric** as an example. You have the Spark engine and the Lakehouse SQL analytics endpoint. Spark comes from the open source distributed computing world, whereas the Lakehouse SQL endpoint inherits over 3 decades of database design, query optimization, and engine maturity from SQL Server. They have completely different architectures and optimization models. Yet from a user perspective, they feel like a single solution.

## How Spark and the SQL endpoint coexist

Despite different engines, if you create a table in Spark using a Fabric Notebook, it automatically appears in the Lakehouse SQL endpoint. How is that possible? Fabric manages this integration through a background process that scans the Lakehouse for changes and updates the SQL endpoint metadata. This allows you to ingest and transform data with Spark, then query the same table using T-SQL. Then connecting tools like Power BI, Tableau, Qlik Sense, or almost any dashboard application becomes straightforward because nearly everything can talk to a SQL endpoint.

When everything works as expected, customers do not think about what is happening behind the scenes, and that is exactly the goal. But sometimes those details surface.

You might create a table in Spark, open the Lakehouse SQL endpoint, and notice that the table does not appear. Or it appears but does not return the data you expect. That is usually when the illusion of a single unified platform starts to break.

- For example, at the time I am writing this post, the background synchronization process only applies metadata changes while the SQL endpoint is active. If the endpoint is inactive and a pipeline makes changes in the Lakehouse, then immediately tries to query those changes through the SQL endpoint, the updates may not be visible yet. Metadata synchronization can take a few seconds or even a few minutes, depending on the situation.

- If you create a table in Spark with more than 1024 columns, it will not appear in the Lakehouse SQL endpoint. Why? Because the SQL engine behind it is based on SQL Server, which has a limit of 1024 columns per table (yes, since SQL Server 2016 we can exceed that limit using [sparse columns](https://learn.microsoft.com/en-us/sql/relational-databases/tables/use-sparse-columns?view=sql-server-ver17), but Lakehouse SQL endpoint does not support sparse columns, at least for now).

- Another common scenario is performance. You read data from a Delta table in Spark and the operation completes very quickly. Then you run a very similar query against the same table using the Lakehouse SQL endpoint, and it takes significantly longer. For many users, this feels inconsistent. If it is the same platform, why is the performance so different? The answer is simple: it is not the same engine. Spark and the SQL endpoint have different execution models, different optimization strategies, and different assumptions. A SQL engine is highly dependent on [statistics](https://learn.microsoft.com/en-us/fabric/data-warehouse/statistics) for cardinality estimation and query planning. If statistics are missing or not representative, performance can degrade. Spark uses a different optimization approach and may behave very differently on the same dataset.

## Conclusion

The beauty of modern data platforms lies in their integration layers. They make multiple engines behave as if they were a single system. Most of the time, this abstraction works extremely well. When it does not, the architectural boundaries suddenly become very real.

Integration makes everything look simple. Under the hood, it is not that simple. The engines optimize differently, they scale differently and have different limitations. 

The more you understand how these engines differ, the easier it becomes to troubleshoot unexpected behavior and design solutions that perform consistently.