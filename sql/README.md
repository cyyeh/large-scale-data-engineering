# [SQL on Big Data](https://hackmd.io/@distributed-systems-engineering/nosql-systems)

###### tags: `data-engineering`

course website: https://event.cwi.nl/lsde/2019/sql.shtml

## Overview

In this lecture we look at two kinds of systems for SQL on Big Data, namely ==SQL-on-Hadoop== systems (most slides) and ==SQL-in-the-cloud== systems (last few slides only).

Database systems we focus here on are for analysis; what traditionally is called ==OLAP (On Line Analytical Processing)==, in SQL terms. In OLAP workloads, users run a few complex read-only queries that search through huge tables. These queries may take a long time. OLAP can be contrasted with ==OLTP (On Line Transactional Processing)==, which is about sustaining a ==high-throughput of single-record read and update queries==, e.g. many thousands per second. OLTP systems also have their Big Data equivalents in the form of NoSQL systems, such as ==HBase== and ==Cassandra==. However, for data science, where we analyze data, we focus on analytical database systems, so noSQL systems are out of scope for this module.

Modern analytical database systems have undergone a transformation in the past decade. The idea that you use the same technology for OLAP and OLTP is gone. Analytical database systems have moved from row-wise to columnar storage and are called =="column stores"==. Column stores typically use ==data compression==; data is ==compressed per column== and this is more effective since the value distribution is more regular than if values from different columns are mixed (as happens in row-stores). Besides general-purpose compression (ZIP, bzip, etc) we discussed a number of database compression schemes that exploit knowledge of the table column datatypes and distributions, among others ==RLE (Run-Length Encoding)==, ==BitMap storage== and ==Differential Encoding== and ==Dictionary Encoding==. Compression helps improve performance because the data becomes smaller and therefore less memory, disk and network traffic is needed when executing queries, often improving performance. ==Sometimes it is even possible to answer queries directly on the compressed data, without decompressing it==; in such cases compression is no longer a trade-off between less data movement for more CPU computation (decompression) but a win-win (less data movement and less computation).

Other storage tricks are =="zone-maps" which keep simple statistics (Min,Max) for large tuple ranges, which allow to avoid reading (skipping) a zone in the table if the WHERE condition ask for values that outside the [Min,Max] range of a zone==. Parallel database systems that run on multiple machines often let the database user (data architect) decide on table distribution (which machine gets which rows?), and partitioning to split the table on each machine further in smaller partitions. ==Distribution (DISTRIBUTE BY) is typically done by "hashing" on a key column== (a hash function is a random but deterministic function) to ensure that data gets spread evenly. ==Partitioning (PARTITION BY) is often done on a time-related column==. One goal of partitioning is to speed up queries by skipping partitions that do not match a WHERE condition. A second goal is data lifecycle management, so one can keep the last X days of data in X partitions, by each day dropping the oldest partition (containing the data of X days ago) and starting a new empty partition in which new data gets inserted. Distribution and partitioning are typically applied both, independently.

Modern analytical database systems also overhaul the SQL query engine, using either ==vectorization or just-in-time compilation (into fast machine code)== to use much less CPU resources per processed tuple than OLTP row-stores. Finally, ==high-end analytical database systems are parallel systems that run on a cluster== and try to involve all cores of all nodes in queries, to make the system faster and more scalable.

Modern Hadoop data formats such as ==ORC== and ==Parquet== have adopted columnar data storage: even though they store all columns together in a HDFS file (to make sure all columns are on the same machine), inside that file the data is placed column after column in huge chunks of rows (rowgroups). Applications that only read a few columns can thus skip over the unused columns. (less I/O). These new data formats also apply columnar compression and sometimes also zone-maps. So, the more advanced SQL-on-Hadoop systems try to adopt all modern ideas on analytical database systems (e.g. ==Hive== uses vectorization, ==Impala== uses JIT), while also trying to fit into the Hadoop ecosystem. That means for instance, that they must try to ensure that they execute data near to where it resides on HDFS (to save bandwidth), to be able to query many different Hadoop data formats in situ, without requiring to first load them into the database, and coordinate their work on the Hadoop cluster (busy cores, RAM requirements) with YARN, in order not to run out of resources and/or suffer from interference by other cluster users.

## References

- Course Materials
    - [SQL on Big Data](https://github.com/cyyeh/large-scale-data-engineering/blob/master/sql/08-SQL%20on%20Big%20Data.pdf)
- Supplementary Materials
    - [Hive: a warehousing solution over a map-reduce framework](https://github.com/cyyeh/large-scale-data-engineering/blob/master/sql/hive.pdf)
    - [Amazon Redshift and the Case for Simpler Data warehouses](https://github.com/cyyeh/large-scale-data-engineering/blob/master/sql/aws-redshift.pdf)
    - [The Snowflake Elastic Data Warehouse](https://github.com/cyyeh/large-scale-data-engineering/blob/master/sql/snowflake.pdf)
    - [presentation on Snowflake](https://github.com/cyyeh/large-scale-data-engineering/blob/master/sql/snowflake-slide.pdf)
    - [Choosing A Cloud DBMS: Architectures and Tradeoffs](https://github.com/cyyeh/large-scale-data-engineering/blob/master/sql/choosing-cloud-dbms.pdf)
    - [Infoworld Article on SQL-on-Hadoop systems](https://www.infoworld.com/article/2683729/10-ways-to-query-hadoop-with-sql.html)
    - [ZDNet article on SQL-on-Hadoop systems](https://www.zdnet.com/article/sql-and-hadoop-its-complicated/)
    - [ZDNet article on SQL-in-the-cloud systems](https://www.zdnet.com/article/cloud-data-warehouse-race-heats-up/)
    - [Introducing the Amazon Redshift data warehouse](https://www.youtube.com/watch?v=JxLpj_TnisM)
    - [Exploring BigData with Google BigQuery](https://www.slideshare.net/DharmeshVaya/exploring-bigdata-with-google-bigquery)