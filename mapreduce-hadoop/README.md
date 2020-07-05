# [MapReduce & Hadoop](https://hackmd.io/@distributed-systems-engineering/mapreduce-hadoop)

###### tags: `data-engineering`

course website: https://event.cwi.nl/lsde/2019/mapreduce.shtml

## Overview

Arguably, ==MapReduce== is what started the whole Big Data wave and what got Google its initial technical advantage. In this lecture we first motivate MapReduce by looking at the difficulties previously experienced (in Super Computing) in developing parallel programs, in terms of programming expertise and debugging effort. We argue that ==for Cluster Computing we need to raise the level of abstraction, viewing the whole cluster as a single machine ("the cluster is the computer"), and developing programming frameworks on that higher level==.

We also take a look at cluster hardware architectures (single node, rack of e.g. 80 nodes, cluster of e.g. 40 racks) and analyze the basic costs to access data from (i) local CPU cache, (ii) local memory, (iii) another computer in the rack, (iv) another computer in the cluster, (v) a computer in a different cluster. (vi) a computer in a different datacenter. ==This shows that disk, and network communication latencies can be very high (and do not improve over time)==, and though bandwidth does improve with newer hardware, it can still be precious. This motivates cluster programs to communicate less, and if at all, in burst, and preferably read data from the local node and if not then at least sequentially and at a large granularity (==idea: minimize bandwidth usage, and amortize latency using large transfers==).

We then describe the MapReduce framework. The framework automatically spreads computation over a cluster, following the above rules: ==it tries to move computation to the data (minimize bandwidth usage)== and when communicating it does so in large transfers. MapReduce relies on a distributed file system named ==HDFS (Hadoop Distributed File System)== that stores files in large blocks of ==64MB== and ==replicates these blocks on three machines (by default)==. To keep replication simple, ==HDFS files cannot be modified, just appended to==. A special master machine runs the ==namenode== which keeps track of all files, their blocks, and on which machines (called ==datanodes==) these are stored. ==HDFS in principle stores all blocks of a HDFS file on the same three datanodes together==, and the datanode which originally wrote the data always is one of them. Though, if a datanodes goes down, the namenode will detect this and replicate the block on another datanode to compensate. The datanodes (tend to) run Linux, and store all HDFS data together as Linux files. But in Linux one cannot see the HDFS filesystem directly, one has to use a special HDFS utility program to browse through it (cd, ls/list, copy files) and see data. So, Hadoop (and HDFS) is a ==cluster software layer on top of Linux==.

The user is expected to write a ==Map()== and ==Reduce()== function, both of which map (key,value) pairs into other (key,value) pairs. Both Map() and Reduce() functions can emit zero or more result pairs for each input pair. The reducer receives as input (key,value*): the second parameter is the full list of all values that were emitted by the mappers for that key - so each key is handled by exactly one Reduce() call. Optionally, a ==Combine()== function may be placed in between Map() and Reduce(), which can reduce the amount of communication between mappers and reducers. Its input and output parameters are the same format as the input of the Reduce() function.

The MapReduce framework optimized data movement by asking the namenode for the location(s) of the HDFS input files and assigning files (or parts of files, called splits) to mappers on machines where this data is on the local disk. Because the reducers get data from all mappers, as determined by the key, ==MapReduce must perform network communication for the reduce phase==. If provided, it will use Combine() functions to reduce the data volume that needs to be sent.
Examples were given in the python programming language (pseudo-code). ==Hadoop is the open-source implementation of MapReduce==. ==The default programming language in Hadoop is Java, but bindings for other languages are available, and python is quite popular==.
In the past 10 years, Hadoop has evolved from a MapReduce clone into a whole ecosystem with many tools. ==This is the most popular "cluster operating system" for organizations that manage their own Big Data clusters ("on premise", as opposed to in the cloud)==. This evolution was enabled by the second major version of Hadoop that separated MapReduce from job scheduling, i.e. ==YARN==. ==This separation of general facilities (storage: HDFS, scheduling: YARN, compute: MapReduce) thus created room for a whole ecosystem== beyond MapReduce, with many alternative tools (e.g. SQL systems such as ==Impala== and ==Hive==) and programming frameworks (e.g. ==Flink== and ==Spark==) operating on top of it.

## Course Materials Summary

### Parallel and Concurrent Programming

![](https://i.imgur.com/JQgjbsQ.png)

- Parallelisation challenges
    - How do we assign work units to workers?
    - What if we have more work units than workers?
    - What if workers need to share partial results?
    - How do we know all the workers have finished?
    - What if workers die?
    - What if data gets lost while transmitted over the network?
- Common theme of all above problems
    - a synchronization issue
        - Communication between workers (e.g., to exchange state)
        - Access to shared resources (e.g., data)
- Managing multiple workers
    - Tools
        - Semaphores (lock, unlock)
        - Conditional variables (wait, notify, broadcast)
        - Barriers
    - Problems
        - Deadlock, livelock, race conditions...
        - Dining philosophers, sleeping barbers, cigarette smokers...
    - Moral of the story: be careful!
- Current tools
    - Programming models
        - Shared memory (pthreads)
            - ![](https://i.imgur.com/CyZXewW.png)
        - Message passing (MPI)
            - ![](https://i.imgur.com/M1kJm16.png)
    - Design patterns
        - Master-slaves
            - ![](https://i.imgur.com/XUlHv1A.png)
        - Producer-consumer flows
            - ![](https://i.imgur.com/uhOK8J3.png)
        - Shared work queues
            - ![](https://i.imgur.com/k3goedt.png)
- The MapReduce Framework alleviates human bottleneck on parallel and concurrent programming
    - It’s all about the right level of abstraction
        - Moving beyond the von Neumann architecture
        - We need better programming models
    - Hide system-level details from the developers
        - No more race conditions, lock contention, etc.
    - Separating the what from how
        - Developer specifies the computation that needs to be performed
        - Execution framework (aka runtime) handles actual execution

---

### MapReduce and HDFS

- Big data needs big ideas
    - Scale “out”, not “up”
        - Limits of SMP and large shared-memory machines
    - Move processing to the data
        - Cluster has limited bandwidth, cannot waste it shipping data around
    - Process data sequentially, avoid random access
        - Seeks are expensive, disk throughput is reasonable, memory throughput is even better
    - Seamless scalability
        - From the mythical man-month to the tradable machine-hour
    - Computation is still big
        - But if efficiently scheduled and executed to solve bigger problems we can throw more hardware at the problem and use the same code
        - Remember, the datacenter is the computer
- MapReduce
    - ![](https://i.imgur.com/8Rwpara.png)
    - Programmers specify two functions
        - `map (k1, v1) → [<k2, v2>]`
        - `reduce (k2, [v2]) → [<k3, v3>]`
            - All values with the same key are sent to the same reducer
    - The execution framework(see MapReduce runtime) handles everything else
    - This is the minimal set of information to provide
    - Usually, programmers also specify
        - `partition (k’, number of partitions) → partition for k’`
            - Often a simple hash of the key, e.g., `hash(k’) mod n`
            - Divides up key space for parallel reduce operations
        - `combine (k’, v’) → <k’, v’>*`
            - Mini-reducers that run in memory after the map phase
            - Used as an optimization to reduce network traffic
- MapReduce runtime
    - Orchestration of the distributed computation
    - Handles scheduling
        - Assigns workers to map and reduce tasks
    - Handles data distribution
        - Moves processes to data
    - Handles synchronization
        - Gathers, sorts, and shuffles intermediate data
    - Handles errors and faults
        - Detects worker failures and restarts
    - Everything happens on top of a distributed file system (more information later)
- MapReduce Implementations
    - Google has a proprietary implementation in C++
        - Bindings in Java, Python
    - Hadoop is an open-source implementation in Java
        - Development led by Yahoo, now an Apache project
        - Used in production at Yahoo, Facebook, Twitter, LinkedIn, Netflix, ...
        - The de facto big data processing platform
        - Rapidly expanding software ecosystem
    - Lots of custom research implementations
        - For GPUs, cell processors, etc.
![](https://i.imgur.com/pDVvEFs.png)
- Distributed file system
    - Do not move data to workers, but move workers to the data!
        - Store data on the local disks of nodes in the cluster
        - Start up the workers on the node that has the data local
    - Why?
        - Avoid network traffic if possible
        - Not enough RAM to hold all the data in memory
        - Disk access is slow, but disk throughput is reasonable
    - Implementations
        - GFS (Google File System) for Google’s MapReduce
        - HDFS (Hadoop Distributed File System) for Hadoop
        - Note: all data is replicated for fault-tolerance (HDFS default:3x)
- GFS
    - Assumptions
        - Commodity hardware over exotic hardware
        - High component failure rates
        - “Modest” number of huge files
        - Files are write-once, mostly appended to
        - Large streaming reads over random access
    - Design Decisions
        - Files stored as chunks
            - Fixed size (64MB)
        - Reliability through replication
            - Each chunk replicated across 3+ chunkservers
        - Single master to coordinate access, keep metadata
            - Simple centralized management
        - No data caching
            - Little benefit due to large datasets, streaming reads
        - Simplify the API
            - Push some of the issues onto the client (e.g., data layout)
- From GFS to HDFS
    - Terminology differences:
        - GFS master = Hadoop namenode
        - GFS chunkservers = Hadoop datanodes
    - Differences
        - Different consistency model for file appends
        - Implementation
        - Performance
- HDFS architecture
    ![](https://i.imgur.com/69joLGz.png)
- Namenode responsibilities
    - Managing the file system namespace
        - Holds file/directory structure, metadata, file-to-block mapping, access permissions, etc.
    - Coordinating file operations
        - Directs clients to datanodes for reads and writes
        - No data is moved through the namenode
    - Maintaining overall health
        - Periodic communication with the datanodes
        - Block re-replication and rebalancing
        - Garbage collection
![](https://i.imgur.com/xzhKHoS.png)

---

### Programming for a Data Center

- Data Center Storage Hierarchy
    ![](https://i.imgur.com/BDRpRSi.jpg)
- Important Latencies
    ![](https://i.imgur.com/ul9MPvH.png)
 
---

### Developing MapReduce Algorithms
    
- MapReduce algorithm design
    - How do you express everything in terms of `map()`, `reduce()`, `combine()`, and `partition()`?
- Computing the mean
    ![](https://i.imgur.com/AmhPBbQ.png)
- Basic Hadoop API
    - Mapper
        - `void setup(Mapper.Context context)`
            - Called once at the beginning of the task
        - `void map(K key, V value, Mapper.Context context)`
            - Called once for each key/value pair in the input split
        - `void cleanup(Mapper.Context context)`
            - Called once at the end of the task
    - Reducer/Combiner
        - `void setup(Reducer.Context context)`
            - Called once at the start of the task
        - `void reduce(K key, Iterable<V> values, Reducer.Context ctx)`
            - Called once for each key
        - `void cleanup(Reducer.Context context)`
            - Called once at the end of the task
- Anatomy of a job
    - MapReduce program in Hadoop = Hadoop job
        - Jobs are divided into map and reduce tasks
        - An instance of running a task is called a task attempt (occupies a slot)
        - Multiple jobs can be composed into a workflow
    - Job submission
        - Client (i.e., driver program) creates a job, configures it, and submits it to jobtracker
        - That’s it! The Hadoop cluster takes over
    - Behind the scenes
        - Input splits are computed (on client end)
        - Job data (jar, configuration XML) are sent to JobTracker
        - JobTracker puts job data in shared location, enqueues tasks
        - TaskTrackers poll for tasks
        - Off to the races
- Input and output
    - InputFormat
        - TextInputFormat
        - KeyValueTextInputFormat
        - SequenceFileInputFormat
    - OutputFormat
        - TextOutputFormat
        - SequenceFileOutputFormat

---

### The Hadoop Ecosystem

![](https://i.imgur.com/1ZcXemn.png)
- YARN: Hadoop version 2.0
    - YARN = Yet-Another-Resource-Negotiator
    - Provides API to develop any generic distribution application
    - Handles scheduling and resource request
    - MapReduce (MR2) is one such application in YARN
- YARN architecture
    ![](https://i.imgur.com/w8PHMbV.png)
- The Hadoop Ecosystem
    - Basic services
        - HDFS = Open-source GFS clone originally funded by Yahoo
        - MapReduce = Open-source MapReduce implementation (Java,Python)
        - YARN = Resource manager to share clusters between MapReduce and other tools
        - HCATALOG = Meta-data repository for registering datasets available on HDFS (Hive Catalog)
        - Spark = new in-memory MapReduce++ based on Scala (avoids HDFS writes)
    - Data Querying
        - Hive = SQL system that compiles to MapReduce(Hortonworks)
        - Impala, or, Drill = efficient SQL systems that do *not* use MapReduce(Cloudera,MapR)
        - SparkSQL = SQL system running on top of Spark
    - Graph Processing
        - Giraph = Pregel clone on Hadoop(Facebook)
        - GraphX = graph analysis library of Spark
    - Machine Learning
        - MLib = Spark –based library of machine learning algorithms

## References

- Course Materials
    - [The MapReduce Framework and Hadoop](https://github.com/cyyeh/large-scale-data-engineering/blob/master/mapreduce-hadoop/03-MapReduce%20%26%20Hadoop.pdf)
- Supplementary Materials
    - [MapReduce: simplified data processing on large clusters](https://github.com/cyyeh/large-scale-data-engineering/blob/master/mapreduce-hadoop/mapreduce.pdf)
    - [The Hadoop Distributed File System (HDFS)](https://github.com/cyyeh/large-scale-data-engineering/blob/master/mapreduce-hadoop/hdfs_design.pdf)
    - [A Very Brief Introduction to MapReduce](https://github.com/cyyeh/large-scale-data-engineering/blob/master/mapreduce-hadoop/map_reduce_tutorial.pdf)
    - [MapReduce according to Wikipedia](https://www.wikiwand.com/en/MapReduce)
    - [Simple introduction to HDFS](http://hadoopilluminated.com/hadoop_illuminated/HDFS_Intro.html)
    - [An Introduction to MapReduce](https://www.slideshare.net/franebandov/an-introduction-to-mapreduce-6789635)
    - [Hadoop Tutorial: Intro to HDFS](https://www.youtube.com/watch?v=ziqx2hJY8Hg)