# [The Spark Framework](https://hackmd.io/@distributed-systems-engineering/spark)

###### tags: `data-engineering`

course website: https://event.cwi.nl/lsde/2019/spark.shtml

## Overview 

In this lecture we focus on the ==Spark== system, which is by far the hottest technology entering the Hadoop ecosystem (though we should mention there is a very similar framework from Europe, called ==Apache Flink==, which is very much alike Spark, and arguably better on some fronts, in particular streaming data). ==Where MapReduce writes all data that Mappers send to Reducers to file first, Spark attempts to avoid doing so, keeping as much data in RAM as possible==. But, since Spark does not provide a distributed filesystem, or a cluster resource manager, ==it effectively depends on HDFS and YARN, making Spark a member of the Hadoop ecosystem==. In the Amazon cloud, we can use Spark via ==EMR== (Elastic MapReduce), which is Amazon's Hadoop with pre-installed Spark. However, it is also possible to use Spark without Hadoop, though one then somehow needs another shared filesystem and job scheduler instead of HDFS and Spark. The makers of Spark (the company Databricks) operate such a Hadoop-less Spark in the Amazon cloud as a service, where the shared storage is provided by Amazon S3 (a similar service in the Microsoft Azure cloud now also exists).

The central concept in Spark is the ==RDD: Resilient Distributed Dataset==. This concept is Spark's answer to ==addressing fault tolerance in distributed computation==. As Spark wants to avoid writing to disk, the question is how fault-tolerance is achieved if a node dies during a computation, and the input was not written to a replicated disk. RDDs are the answer: ==they are either persistent datasets (on HDFS) or declarative recipes of how to create a dataset from other RDDs==. This recipe allows Spark to recompute lost (parts of) RDDs of failed jobs on-the-fly.

Spark is a programming framework that offers Map and Reduce, but ==it offers 20+ more communication patterns==, e.g. also joins between datasets. As such, it is a ==generalization of MapReduce==. Also notable is its focus on ==functional programming==, in particular ==Scala==. Functional programming languages have ==no side-effects== and therefore are much easier to parallelize than traditional programming languages. This is what Spark does: automatically optimize functional programs over a cluster infrastructure. One feature of functional programming that is often used in Spark are ==lambdas: these are implicit function definitions==. The functions you would write in MapReduce for Map and Reduce in Java, you can provide shortened, inline, as lambdas in Scala. Typically, such a lambda is a parameter to the high-level operators of Spark, e.g. a join operator would expect a boolean function that defines when two items from the two RDDs being joined, match.

Newer versions of Spark introduced ==DataFrames==. A DataFrame is a conceptual layer on top of RDDs. Essentially, ==a DataFrame is a table, with column names and types==. Thanks to this extra information, Spark can execute queries on DataFrames more efficiently than queries on RDDs because it can optimize these queries. There is a query optimizer in Spark, called ==Catalyst==. Both Spark SQL and all scripts on DataFrames use this optimizer. Spark was further made to perform better by low-level compilation of Spark queries into java code, and also ==support columnar-storage better==.

Spark is a distributed Big Data processing environment ==using Scala, Java or Python programs==, but it also has multiple higher-level components, notably ==Spark SQL== (distributed columnar database, will be discussed later), ==GraphX== (vertex-centric graph processing), and ==MLLIB== (machine learning library). The latter two are summarized as follows.

GraphX is a Spark API for ==iterative graph analytics== algorithms on Hadoop. Avoiding the materialization on disk (something that MapReduce would do) is a strength of Spark and it pays off especially in iterative algorithms. The computational model of GraphX is based on the concept of "==Think like a Vertex==" originally proposed in the Pregel system by Google (not open-source). Beyond Spark GraphX, other popular such think-like-a-vertex systems are ==Apache Giraph== and ==Flink Gelly==. In all these systems, the user writes a method that describes how one vertex reacts on messages that come in for it, uses these and its own state to compute its new state and possibly send output messages (to its neighbours over edges). This kind of computation operates in "supersteps": the vertexes receive all messages sent to them in the previous superstep, do their computation and sending and then wait until all other vertexes have done so. Then the system moves to the next superstep, and this keeps iterating until a fixed number of iterations or until no vertex sends any message anymore in a superstep. This relatively simple computational model is illustrated to be quite powerful, Google's PageRank can be implemented in just a few lines of code.

MapReduce and Spark have been criticized for their low absolute performance (the former more so than the latter). Concretely, the COST paper (referenced below) argues that ==when presenting a new parallel system or algorithm, one should always compare with an optimized sequential (non-parallel) baseline and compute how many cores/computers one would need to equal its performance (this is the metric COST proposes)==. While Spark is an improvement over MapReduce, and its use of JIT compiled query plans (see: SQL on Big Data lecture) have improved its absolute performance further, there is still a gap with what could be achieved in terms of performance. In terms of our practicum1, this is evidenced by the relatively slow performance even optimized assignment 1c solutions achieve over the native implementations of the marketing query in C/C++.

“You can have a second computer once you’ve shown you know how to use the first one.” –Paul Barham

Spark offers its MLLIB library that provides a set of ready-to-run machine learning algorithms that work on RDDs (and now also on DataFrames), and thus scale on a cluster. We focus on its ==Alternating Least Squares (ALS)== implementation for ==collaborative filtering==. In collaborative filtering, typically we have a very partial set of datapoints that describe users and their interests (e.g. songs). The purpose of collaborative filtering is to derive what all users think of all possible interests, specifically to recommend the most interesting items (songs) to users. ALS models this as a matrix problem where for each user and for each song we keep a vector of numbers: each element in this vector corresponds to a latent factor (characteristic). If we would know these factors, then multiplying them would give the matrix of all user/song interests. The ALS method iteratively approaches optimal vector values, using a (possibly small) set of known interests as training set.

MLLIB contains many different machine learning algorithms that can be easily connected to data through the Spark concept of RDDs. The output of the learning (the model) is also an RDD and can subsequently be used to score new users (i.e. make recommendations). Besides tight integration with Spark processing pipelines (and e.g. Spark SQL) another advantage of MLLIB is that the algorithms in it are implemented in a scalable, distributed fashion. Therefore, one can address large problems with MLLIB using clusters, which would not fit on a single machine.

## Courses Materials Summary

### Spark/Spark SQL

- What is Spark?
    - Fast and expressive cluster computing system interoperable with Apache Hadoop
    - Improves efficiency through(Up to 100× faster)
        - In-memory computing primitives
        - General computation graphs
    - Improves usability through(Often 5× less code)
        - Rich APIs in Scala, Java, Python
        - Interactive shell
- The Spark Stack
    - More details: https://amplab.cs.berkeley.edu/
    ![](https://i.imgur.com/JQwNvf7.png)
- Why a New Programming Model?
    - MapReduce greatly simplified big data analysis
    - But as soon as it got popular, users wanted more
        - More complex, multi-pass analytics (e.g. ML, graph)
        - More interactive ad-hoc queries
        - More real-time stream processing
    - All 3 need faster data sharing across parallel jobs
- Data Sharing in MapReduce
    ![](https://i.imgur.com/CJASZw1.png)
    -  Slow due to replication, serialization, and disk IO
- Data Sharing in Spark
    ![](https://i.imgur.com/H6WrTaM.png)
    -  ~10× faster than network and disk
- Spark Programming Model
    - Key idea: resilient distributed datasets (RDDs)
        - Distributed collections of objects that can be cached in memory across the cluster
        - Manipulated through parallel operators
        - Automatically recomputed on failure
    - Programming interface
        - Functional APIs in Scala, Java, Python
        - Interactive use from Scala shell
![](https://i.imgur.com/iO9oFyS.png)
- Spark in Scala and Java
    ![](https://i.imgur.com/txGZehs.png)
- Supported Operators
    ![](https://i.imgur.com/WWLsaMI.png)
- Software Components
    ![](https://i.imgur.com/BcIoy6v.png)
    - Spark client is library in user program (1 instance per app)
    - Runs tasks locally or on cluster
        - Mesos, YARN, standalone mode
    - Accesses storage systems via Hadoop InputFormat API
        - Can use HBase, HDFS, S3, ... 
- Task Scheduler
    ![](https://i.imgur.com/17XIQ5W.png)
    - General task graphs
    - Automatically pipelines functions
    - Data locality aware
    - Partitioning aware to avoid shuffles
- Spark SQL
    - Columnar SQL analytics engine for Spark
        - Support both SQL and complex analytics
        - Columnar storage, JIT-compiled execution, Java/Scala/Python UDFs
        - Catalyst query optimizer (also for DataFrame scripts)
- Hive Architecture
    ![](https://i.imgur.com/rqH60NP.png)
- Spark SQL Architecture
    ![](https://i.imgur.com/9jWZgSb.png)
- From RDD to DataFrame
    - A distributed collection of rows with the same schema (RDDs suffer from type erasure)
    - Can be constructed from external data sources or RDDs into essentially an RDD of Row objects (SchemaRDDs as of Spark < 1.3)
    - Supports relational operators (e.g. where, groupby) as well as Spark operations.
    - Evaluated lazily -> non-materialized logical plan
- DataFrame: Data Model
    - Nested data model
    - Supports both primitive SQL types (boolean, integer, double, decimal, string, data, timestamp) and complex types (structs, arrays, maps, and unions); also user defined types.
    - First class support for complex data types
- DataFrame Operations
    - Relational operations (select, where, join, groupBy) via a DSL
    - Operators take expression objects
    - Operators build up an abstract syntax tree (AST), which is then optimized by Catalyst.
    - Alternatively, register as temp SQL table and perform traditional SQL query strings
- Catalyst: Plan Optimization & Execution
    ![](https://i.imgur.com/yRDJX68.png)
- Performance
    ![](https://i.imgur.com/sAkLwho.png)

### GraphX/GraphFrames

- Pregel: computational model
    - Based on Bulk Synchronous Parallel (BSP)
        - Computational units encoded in a directed graph
        - Computation proceeds in a series of supersteps
        - Message passing architecture
    - Each vertex, at each superstep
        - Receives messages directed at it from previous superstep
        - Executes a user-defined function (modifying state)
        - Emits messages to other vertices (for the next superstep)
    - Termination
        - A vertex can choose to deactivate itself
        - Is “woken up” if new messages received
        - Computation halts when all vertices are inactive
- Pregel: implementation
    - Master-Slave architecture
        - Vertices are hash partitioned (by default) and assigned to workers
        - Everything happens in memory
    - Processing cycle
        - Master tells all workers to advance a single superstep
        - Worker delivers messages from previous superstep, executing vertex computation
        - Messages sent asynchronously (in batches)
        - Worker notifies master of number of active vertices
    - Fault tolerance
        - Checkpointing
        - Heartbeat/revert
- Graph Programming Frameworks
    - Google Pregel
        - Non open-source, probably not used much anymore
    - Apache Giraph
        - Developed and used by Facebook
    - Apache Flink
        - Gelly API
    - Apache Spark
        - GraphX API
        - DataFrames API

### MLlib

- What is MLLIB?
    - MLlib is a Spark subproject providing machine learning primitives
        - initial contribution from AMPLab, UC Berkeley
        - shipped with Spark since version 0.8
    - Algorithms
        - classification: logistic regression, linear support vector machine (SVM), naive Bayes
        - regression: generalized linear regression (GLM)
        - collaborative filtering: alternating least squares (ALS)
        - clustering: k-means
        - decomposition: singular value decomposition (SVD), principal component analysis (PCA)

## References

- Course Materials
    - [Spark](https://github.com/cyyeh/large-scale-data-engineering/blob/master/spark/04-The%20Spark%20Framework.pdf)
- Supplementary Materials
    - [Spark: Cluster Computing with Working Sets](https://github.com/cyyeh/large-scale-data-engineering/blob/master/spark/spark.pdf)
    - [Scalability! But at what COST?](https://github.com/cyyeh/large-scale-data-engineering/blob/master/spark/cost.pdf)
    - [Apache Spark: A Unified Engine for Big Data Processing](https://github.com/cyyeh/large-scale-data-engineering/blob/master/spark/spark-cacm.pdf)
    - [Clash of the Titans: MapReduce vs. Spark for Large Scale Data Analytics](https://github.com/cyyeh/large-scale-data-engineering/blob/master/spark/mapreduce-vs-spark.pdf)
    - [Pregel: A System for Large-Scale Graph Processing](https://github.com/cyyeh/large-scale-data-engineering/blob/master/spark/pregel.pdf)
    - [MLI: An API for Distributed Machine Learning](https://github.com/cyyeh/large-scale-data-engineering/blob/master/spark/mli.pdf)
    - [Matrix Factorization Techniques for Recommender Systems](https://github.com/cyyeh/large-scale-data-engineering/blob/master/spark/matrixfactorization.pdf)
    - [Powering Data Science with Spark - Ion Stoica (Databricks) Ali Ghodsi (Databricks)](https://www.youtube.com/watch?v=GuVvNjZaxTs)
    - [Structuring Apache Spark 2.0: SQL, DataFrames, Datasets And Streaming - by Michael Armbrust](https://www.youtube.com/watch?v=1a4pgYzeFwE)
    - [Collaborative Filtering with Spark](https://www.slideshare.net/MrChrisJohnson/collaborative-filtering-with-spark)
    - [Big Data Processing with Apache Spark](https://www.infoq.com/articles/apache-spark-introduction/)
    - [Introducing DataFrames in Apache Spark for Large Scale Data Science](https://databricks.com/blog/2015/02/17/introducing-dataframes-in-spark-for-large-scale-data-science.html)
    - [How to install PySpark and Jupyter Notebook in 3 Minutes](https://www.sicara.ai/blog/2017-05-02-get-started-pyspark-jupyter-notebook-3-minutes)