---
title: "Reads in Cassandra"
date: 2023-02-08T11:36:20-06:00
draft: false
tags: ["Databases", "Software Engineering", "System Design", "Cassandra"]

ShowToc: true
TocOpen: true 
---

Welcome to my technical blog. In this blog, we will dive into the inner workings of Cassandra, the popular, high-performance, and highly-scalable distributed database system. We will explore how it handles read requests from clients. We will cover topics such as how Cassandra achieves its high level of availability, how it routes read requests to the appropriate node, and how it handles situations where a node does not have a replica of the requested data. By the end of this blog, you will have a solid understanding of how reads work in Cassandra and how it helps to ensure a fast and reliable experience for users. So, let's get started!

Cassandra enables clients to access data from any node, even if that node doesn't have a replica of the requested data. In these instances, the node without the data acts as a coordinator, retrieving the data from nodes that do have it. Cassandra reads can be slower than writes because of file I/O, particularly when data needs to be fetched from disk. However, read performance can be improved through caching, increasing the number of nodes, and using node memory to read the data. As a result, data can be retrieved and returned to the client in Cassandra either from the cache, memory, or disk.

### Data storage in Cassandra

Cassandra uses LSM (Log-structured Merge Trees) to persist its data, which is similar to databases such as RocksDb and LevelDb. This is different from traditional RDBMSs, which typically use a B-tree implementation. To ensure durability, Cassandra also writes data to a commit log, similar to a Write-Ahead Log (WAL) and writes to memory as well.

Each table in Cassandra has an associated in-memory table. As data is written to the commit log in a sequential order, it is sorted by the partition key and clustering key and stored within the in-memory table. The partition key determines which node the data is stored on, and the clustering key determines the order in which it is stored on that node. The primary key for Cassandra is effectively composed of the partition key and the clustering key. When the in-memory table reaches its optimal size, determined by the chunk size of the data, it is saved to disk as an SSTable (Sorted String Table). This table contains a data file and an index file, both of which include in-memory indexes. Regular compaction is performed by merging and rewriting SSTables which reduces disk space and improves read performance by removing outdated data and consolidating it into fewer, larger SSTables. 

Data retrieval on a single node follows a similar path with a few optimisations.

### Read path on a single node

![cassandra-single-node-read](https://imgur.com/pJWms3I.png)

### Reading from Cache

Cassandra uses several types of caches to improve read performance.

* Row Cache: This cache stores the most recent data for a table in memory, allowing for fast access to frequently-accessed data. It is useful for read-heavy workloads and can take considerable amount of space. 

* Key Cache: This cache stores the keys of recently-accessed rows in memory, allowing for faster lookups. It is useful for write-heavy workloads. In essence, it is a cache of the PartitionIndex file for a table which is a map of partition keys to their SSTable offsets.

* Counter Cache: This cache stores the most recent counter values in memory, improving the performance of counter-related operations.

* File System Cache: This cache stores data read from disk in memory, allowing for faster access to data that is not already stored in the row or key cache.

All of these caches can be configured with different sizes, depending on the available memory and the specific use case. The caches can also be disabled if not needed.

It's important to note that caching in Cassandra is optional, and enabling it depends on the use case and the amount of memory available. Cassandra distributes the cache data across the cluster. When a node goes down , the client can read from a cache replica. The distributed cache in cassandra is saved periodically into disk which helps read data into cache on start up.  Also, one of the important point is that, if the data is changing frequently, caching may not always improve performance as it will store stale data.

### Reading from Disk

Cassandra reads data from disk using the SSTable data structure. The SSTables are organized on disk as a set of data files, each containing a portion of the data stored in Cassandra. SSTables are immutable, meaning that once written, they are not updated. Instead, when data changes, a new SSTable is written to disk, allowing Cassandra to maintain an ordered view of the data.

The choice of SSTable as a data structure helps improve sequential access of data by leveraging its design as a sorted and indexed file. When reading data, Cassandra performs a binary search on the SSTable index, which is stored in memory, to quickly locate the on-disk position of the desired data. From there, Cassandra can perform a sequential read of the data, since the SSTable data is stored in sorted order on disk.

This design allows Cassandra to perform highly optimized and efficient disk access, enabling it to handle very large data sets with low latency. Additionally, because SSTables are immutable, they can be safely and efficiently compressed, further reducing disk usage and improving performance.:

The partition index and partition index summary files are two key components in Cassandra's data storage architecture.

The partition index is a file that stores the metadata for each partition of data stored in an SSTable. This metadata includes the starting and ending row keys for each partition, as well as the on-disk position of the data. The partition index is used by Cassandra to efficiently locate partitions when reading data from disk.

The partition index summary file is a smaller, compressed version of the partition index. It contains a sample of the entries from the full partition index, rather than all of them. This allows Cassandra to quickly locate the on-disk position of the data for a given partition, without having to read the full partition index. The partition index summary file is loaded into memory, making it readily available for fast lookups.

Together, the partition index and partition index summary files form a key part of Cassandra's data storage architecture, enabling it to efficiently locate and access data on disk, even in very large data sets.

The below diagram shows a partition Index file and its corresponding data file.

![cassandra-partion-index-file](https://i.imgur.com/OvJlmoz.png)

and the relationship between Partition Index Summary File and Partition Index file is shown below

![cassandra-partition-index-summary-file](https://i.imgur.com/SQDtdbv.png)

### Summary

* Cassandra being a distributed data base with multiple replicas needs to optimise the read path to be performant
* A read query that comes to a node without data will act as a co-ordinator node and request data from the adjacent nodes. The read consistency setting will have an impact on the overall read performance.
* On a node data can be retrieved either from cache or from disk
* There are many types of cache available are row-cache, key-cache, counter-cache and file cache.
* If the data is not found on cache, then an in memory bloom filter is checked to see if the data exists on the disk.
* If data exists on disk, then the partition index summary file is used to identify the offset on the partition index file.
* The byte offset of the partition key is fetched from the partition index file using the offset from the partition index summary file. OS caching of the partition index file can further improve the seek performance.
* Data is read from the data file using the byte offset of the partition key.

