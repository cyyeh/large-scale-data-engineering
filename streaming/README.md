# [Data Streams](https://hackmd.io/@distributed-systems-engineering/data-streams)

###### tags: `data-engineering`

course website: https://event.cwi.nl/lsde/2019/streaming.shtml

## Overview

A data stream is a ==real-time, continuous, ordered (implicitly by arrival time of explicitly by timestamp) sequence of items==. It is impossible to control the order in which items arrive, nor it is feasible to locally store a stream in its entirety. Stream data management is important in telecommunications, real-time facilities monitoring, stock monitoring and trading, preventing fraudulent transactions and click stream analysis.

To process streams more easily, ==Data Stream Management Systems (DSMS)== have been proposed like ==StreamBase==. ==In DSMS, many things are just opposite from a Database Management System (DBMS)==: in a DBMS the data is persistent and the query volatile, while in a DSMS the query is persistent and gives continuous answers, while the data is volatile. A DSMS processes queries over a stream of data, ==by partitioning that stream in windows and evaluating the query for every new window==, producing a never ending stream of results. The windows can be time-limited, size-limited, or punctuated by specific kinds of events.

Twitter has built an open-source data stream management system called ==Storm==. Storm makes it easy to reliably process unbounded streams of data, doing for realtime processing what Hadoop did for batch processing. With Storm, one creates event workflows, called "==Topologies==" where events stem from stream sources called "==Spouts==" (e.g. Twitter feeds, stock tickers, RSS) and are processed by so-called "==Bolts==". Topologies determine how many of each of such elements are used (each becomes a process on a compute node) and how data is distributed and split between them, allowing all these elements to run in parallel on a large cluster.

The term ==Lambda Architecture== was coined for systems that consist of two Big Data pipelines: one batch-oriented (e.g. MapReduce) that computes static views on most data, and a second pipeline that computes dynamic data views on the last few hours of data using a stream engine (e.g. Storm). This allows to compute current query answers over all data relatively cheaply -- however the architecture duplicates logic in two pipelines. The name Lambda Architecture refers to computing a function over the data; this ability to recompute everything over independent subsets of data makes the architecture possible,

Finally, we discuss the new ==Google DataFlow== which will enter open source life as ==Apache Beam==. This is Google' own successor to MapReduce. It has a data processing framework with operators similar to Map (==ParDo==) and Reduce (==GroupByKey==) - but also many operator such as ==Filter== and ==Join==. These operators can be stitched together in ''pipelines'' and the framework was initially called ==FlumeJava==. ==Dataflow adds to this a stream extension that moves from (Key,Value) pairs used in MapReduce to (Key,Value,EventTime,Window) tuples, where each key-value pair also carries a time when the event occurred, and a session window==. The DataFlow framework allows many times of window specification and allows to explicitly define computation policies based on the EventTime and Processing Time, which are independent concepts. As such, it provides a richer expressive power in to specify temporal behavior than most other streaming engines.

## References

- Course Materials
    - [Data Streams](https://github.com/cyyeh/large-scale-data-engineering/blob/master/streaming/06-DataStreams.pdf)
- Supplementary Materials
    - [Twitter Heron: Stream Processing at Scale](https://github.com/cyyeh/large-scale-data-engineering/blob/master/streaming/twitter-heron.pdf) (and why it is replacing the Storm system)
    - [Discretized streams: Fault-tolerant streaming com-putation at scale](https://github.com/cyyeh/large-scale-data-engineering/blob/master/streaming/spark_streaming.pdf) (Spark Streaming)
    - [The Dataflow Model: A Practical Approach to Balancing Correctness, Latency, and Cost in Massive-Scale, Unbounded, Out-of-Order Data Processing](https://github.com/cyyeh/large-scale-data-engineering/blob/master/streaming/dataflow-model.pdf) (Google Cloud DataFlow)
    - [Approximate Frequency Counts over Data Streams](https://github.com/cyyeh/large-scale-data-engineering/blob/master/streaming/approximate-freq-counts.pdf) (Google Cloud DataFlow)
    - [Storm Tutorial](http://storm.apache.org/releases/current/Tutorial.html)
    - [A Guide To Spark Streaming](https://opensource.com/business/15/4/guide-to-apache-spark-streaming)
    - [Blogpost of Google DataFlow developers comparing it with Spark (Streaming)](https://cloud.google.com/dataflow)
        - note: Google DataFlow is being open sourced as [Apache Beam](https://beam.apache.org/).
    - [GOTO 2012 • Runaway Complexity in Big Data Systems...and a Plan to Stop it • Nathan Marz](https://www.youtube.com/watch?v=ucHjyb6jv08)
    - [Deep Dive with Spark Streaming - Tathagata Das - Spark Meetup 2013-06-17](https://www.slideshare.net/spark-project/deep-divewithsparkstreaming-tathagatadassparkmeetup20130617)
    - [Google Cloud Dataflow - Two worlds become a much better one](https://www.youtube.com/watch?v=I9pR9fvPX10)
    - [Hadoop Summit Europe 2014: Apache Storm Architecture](https://www.slideshare.net/ptgoetz/storm-hadoop-summit2014)