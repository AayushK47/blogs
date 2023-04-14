---
title: "From Relational to Analytical: The Power of Redshift Data Warehousing and Analytics"
datePublished: Fri Apr 14 2023 12:15:55 GMT+0000 (Coordinated Universal Time)
cuid: clggijayl000709msb8u99qmr
slug: from-relational-to-analytical-the-power-of-redshift-data-warehousing-and-analytics
tags: databases, redshift, data-warehousing, relational-database, data-analytics

---

I recently got my hands dirty working on data warehousing and found myself wondering why traditional databases like Postgres aren't commonly used for this task. After some research and experimentation, I discovered that specialized data warehousing solutions like Amazon Redshift offer distinct advantages over regular databases when it comes to handling large volumes of data and supporting complex analytical queries. In this blog post, we'll explore the reasons why Redshift has become the go-to choice for data warehousing and why it's worth considering for your own data storage and analysis needs.

## A Brief Intro to Data Warehousing

Data is the most critical asset for all modern businesses. It is essential for making an informed decision and staying ahead of the competition. The data can be collected from various sources. For example, an e-commerce site can collect data from its website, along with some of its affiliate websites. Now, it's obvious that you don't want to go to each of these sources and analyze data repeatedly.

Enter Data Warehousing.

Data warehousing involves several processes that enable organizations to collect, store, and manage data in a centralized repository. Typically, it starts with data being extracted from various sources, such as transactional databases, log files, and third-party applications. The data collected from all these sources may not necessarily be uniform, so it needs to be transformed and cleaned. This process is known as Extract, Transform, and Load (ETL), and there are many tools available that can be used for this task, such as Talend, Informatica, and AWS Glue. Once the data is transformed and loaded into the centralized data warehouse, it is organized and structured in a way that enables efficient querying and analysis.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1681474289437/936225e2-bca9-44f9-b99e-d8f6b266a2fb.png align="center")

Once the data is loaded into a data warehouse, the next step in the process is to perform analysis and prepare visualizations using the data. Tools like Tableau, Power BI, and Quicksight can be used for this.

Now that you understand what data warehousing is, let's understand why it's not a good idea to use relational databases as the central repository.

## The Limits of Relational Databases for Data Warehousing

Relational Databases are great. I mean they are still used by almost every system in the world to store their data. But when it comes to data warehousing, the size of the data we are talking about is HUGE!!.

Scaling out a relational database is challenging. But let's say that we planned and scaled out our database, we may still be facing a lot of issues in the query performance.

Relational databases are designed to store data in tables with relationships between them. This is great for transactional processing where small amounts of data are frequently accessed and updated. But for analyzing large amounts of data, we require complex queries that involve aggregating, joining and filtering large tables. On top of that, RDBs ensure have to ensure that the data is consistent, which further adds overhead to the query performance. We can employ some strategies to improve the performance, but wouldn't it be easier to just use a solution that is designed to handle complex queries and large data?

Some of the most popular technologies that are used for these purposes include AWS Redshift and Google's Big Query. Let's discuss a bit about the architecture of Redshift and see how it is designed to handle fetching large amounts of data very quickly.

## Unpacking Redshift

Let's unpack Redshift feature by feature and see how each feature helps makes it great for data warehousing and analytics.

### Massively Parallel Processing (MPP) Architecture

The entire service of Redshift is built on a distributed, massively parallel processing (MPP) architecture, which enables it to handle large amounts of data quickly and efficiently.

Each redshift cluster consists of a leader node and multiple compute nodes. The job of leader node is to receive queries for the client, create an execution plan and distribute the compute nodes. Compute node, on the other hand, is where actual data processing occurs. Each compute node consists of one or more slices. A slice is a self-contained processing unit that can execute queries independently and in parallel with other slices in the same node. This distributed architecture is one of the few features used to accelerate the performance of redshift.

![Redshift's Massive Parallel Processing (MPP) Architecture](https://cdn.hashnode.com/res/hashnode/image/upload/v1681473548994/64318e42-916e-4c04-9b38-ecbf8d52be87.png align="center")

### Columnar Database

Under the hood, redshift uses the Postgres database, which brings all the goodness of relational databases that we love. But it is not a relational database, rather it is a columnar database. A columnar database stores all the column-specific data together. This means they store column-specific data together instead of row-specific data. Believe it or not, this helps in faster data retrieval since only the columns required by the query are read, instead of reading the entire row. Also, since the columns store similar data, columnar databases can compress data more efficiently, which results in reduced disk I/O operations and an increase in in-memory operations.

### Zone Maps

A zone map is a metadata stored for each column. Essentially, it is an index that stores the minimum and maximum values of each column. By storing this information, a zone map can help in determining which zone has to be scanned to satisfy the query optimally.

### Sort and Distribution Keys

It's quite obvious that sorted data is much more easily accessible than unsorted data. Sort Keys are used in determining the physical order in which the data is stored. These keys could be defined as a single column or as a compound key (multiple columns).

Distribution keys on the other hand determine how the data is distributed across the nodes in a redshift cluster. As the data is distributed evenly across the nodes, it becomes easier and faster to query data in parallel.

### Materialized View

A materialized view is a precomputed table for storing the results of complex queries. When a materialized view is created, the result of the underlying query is stored in the table. When a query with a similar pattern runs the next time, the data stored in the precomputed table is used instead of running the entire query again. This significantly reduces the query performance, especially for queries that involve a huge amount of data and complex operations.

On top of all the above features, redshift is secure, scalable, easy to use and integrate with BI tools.

## Conclusion

While traditional relational databases like Postgres are great for transactional processing, they may not be the best fit for data warehousing due to their limitations in handling complex queries and large volumes of data.

This is where specialized solutions like Amazon Redshift come in. With its massively parallel processing architecture, columnar database, zone maps, sort and distribution keys, and materialized views, Redshift provides a powerful and efficient platform for data warehousing and analytics.

Hopefully, this blog post has given you a better understanding of data warehousing, where traditional solutions fail and how solutions like Redshift overcome them. If you have any questions or comments, feel free to leave them below!