# [Cloud Computing](https://hackmd.io/@distributed-systems-engineering/cloud-computing)

###### tags: `data-engineering`

course website: https://event.cwi.nl/lsde/2019/cloud.shtml

## Overview 

In this lecture we first define Big Data in terms of data management problems with the ==three V=='s: (i) Volume: the data is large, (2) Variety: the data is often not clean and tabular, but messy (text, even images), (3) Velocity: new data keeps arriving continuously. We also discuss ==Power Laws== in many Big Data problems: the mass of data is in the long tail, the long tail cannot be ignored as it may represent the majority of all datapoints. Power Law distributions are typical in social networks, e.g. the distribution of amount of twitter followers has a power law distribution.

We explain ==Eric Brewers CAP theorem==: a global software system cannot achieve all three of ==Consistency== (reads always reflect the latest updates), ==Availability== (the system is always up), ==Partitionability== (the system is resistant against loss of communication between datacenters), and describe concepts as replication and sharding and their consequences in terms of read speed, update speed and consistency.
As we work on large computer infrastructures, we distinguish between three related areas: (1) ==Super Computing==, where performance is king and programmability a side-issue, (2) ==Cluster Computing==, which is about quickly getting things done on large clusters of unreliable machines, and (3) ==Cloud Computing==, where computation is performed in large clusters operated by third party, sold as a commodity service to users based on their actual use, seamlessly allowing to get more or less of it based on their needs (elasticity). In the practicum where we use ==Hadoop== (cluster computing) on ==Amazon Web Services== (cloud computing) we hence combine (2) and (3). Super computing is out of scope for this module. In cloud computing, we further distinguish between ==IaaS==: Infrastructure-as-a-Service (virtual machines for rent, e.g. Amazon EC2), ==PaaS==: Platform as-a-service (database system for rent, e.g. Amazon Redshift) and ==SaaS==: Software-as-a-Service (application for rent, e.g. Microsoft Office 365 or Salesforce).

The last part of the lecture describes Amazon Web Services (AWS) in some detail to prepare you for the practicum. Services to remember are ==S3== block storage (infinitely scaling storage -- presumably implemented by spreading storage with replication over the hundreds of thousands of machines Amazon owns), ==EBFS== filesystems which are virtual disks that store their data remotely on S3 but do a lot of caching to avoid network traffic, and the ==EC2== service that allows to power up virtual machines in the cloud (and some of the options to choose from in terms of CPU, RAM, and local disks aka "ephimeral storage" which are empty when you start-up).

## Course Materials Summary

### Course Introduction

- The goal of the course is to gain insight into and experience in using hardware infrastructures and software technologies for analyzing ‘big data’.
- Solving such tasks(data management tasks where problem size/complexity requires using a cluster) requires
    - insight in the main factors that underlie algorithm performance
        - access pattern, hardware latency/bandwidth
    - these factors guided the design of current Big Data infratructures
        - helps understanding the challenges
- Understanding the concepts
    - hardware
        - What components are hardware infrastructures made up of? 
        - What are the properties of these hardware components?
        - What does it take to access such hardware?
    - software
        - What software layers are used to handle Big Data?
        - What are the principles behind this software?
        - Which kind of software would one use for which data problem?
- Scientific paradigms
    1. Observing
    2. Modeling
    3. Simulating
    4. Collecting and Analyzing Data
- Big Data Challenges
    - Volume: data larger than a single machine (CPU,RAM,disk)
        - Infrastructures and techniques that scale by using more machines
        - Google led the way in mastering “cluster data processing”
    - Velocity: endless stream of new events
        - No time for heavy indexing (new data keeps arriving always)
        - led to development of data stream technologies
    - Variety: Dirty, incomplete, inconclusive data (e.g. text in tweets)
        - Semantic complications
            - AI techniques needed,not just database queries
        - Technical complications
            - Skewed Value Distributions and “PowerLaws”
            - Complex Graph Structures -> Expensive Random Access
            - Complicates cluster data processing(difficult to partition equally)
            - Localizing data by attaching pieces where you need them makes Big Data even bigger
- Supercomputing vs Cluster Computing vs Cloud Computing
- Economics of Cloud Computing
    - A major argument for Cloud Computing is pricing
        - We could own our machines
        - Since machines rarely operate at more than 30% capacity, we are paying for wasted resources
    - Pay-as-you-go rental model
    - No other costs
    - This makes computing a commodity

### Cloud Computing

- What?
    - Computing resources as a metered service (“pay as you go”)
    - Ability to dynamically provision virtual machines
- Why?
    - Cost: capital vs. operating expenses
    - Scalability: “infinite” capacity
    - Elasticity: scale up or down on demand
- Does it make sense?
    - Benefits to cloud users
    - Business case for cloud providers
- Everything as a service
    - Utility computing = Infrastructure as a Service (IaaS)
    - Platform as a Service (PaaS)
    - Software as a Service (SaaS)
- Several Historical Trends
    - Shared Utility Computing
    - Data Center Co-location
    - Pay as You Go
    - Virtualization
    - Minicomputers
    - The age of the x86 PC
    - Mobile and Server separate
![](https://i.imgur.com/8yFDQop.png)
    - Client requests are handled in the first tier
    - Much use of caching: the second tier
- Eric Brewer’s CAP theorem
    - Consistency & Availability & Partition-tolerance
    - weaken consistency for faster response
- CASE STUDY: AMAZON WEB SERVICES
    - Global Physical Infrastructure
    ![](https://i.imgur.com/TafEHqx.png)
    - EC2 Architecture
    ![](https://i.imgur.com/5EECUK1.png)
    - Simple Storage Service (S3)
        - A bucket is a container for objects and describes location, logging, accounting, and access control. A bucket can hold any number of objects, which are files of up to 5TB. A bucket has a name that must be globally unique.
        - A bucket has a flat directory structure (despite the appearance given by the interactive web interface.)
        - Fundamental operations
            - POST a new object or update an existing object.
            - GET an existing object from a bucket.
            - DELETE an object from the bucket.
            - LIST keys present in a bucket, with a filter.
        - Weak Consistency Model
    - Elastic Block Store
        - An EBS volume is a virtual disk of a fixed size with a block read/write interface. It can be mounted as a filesystem on a running EC2 instance where it can be updated incrementally. Unlike an instance store, an EBS volume is persistent.
        - Fundamental operations
            - CREATE a new volume (1GB-1TB)
            - COPY a volume from an existing EBS volume or S3 object.
            - MOUNT on one instance at a time.
            - SNAPSHOT current state to an S3 object.
        - EBS is approx. 3x more expensive by volume and 10x more expensive by IOPS than S3
        - File-system (EBS and also HDFS) is an abstraction on top of S3
    - Use Glacier for Cold Data
        - Glacier is structured like S3: a vault is a container for an arbitrary number of archives. Policies, accounting, and access control are associated with vaults, while an archive is a single object.

## References

- Course Materials
    - [Course Introduction](https://github.com/cyyeh/large-scale-data-engineering/blob/master/cloud-computing/00-Intro.pdf)
    - [Cloud Computing](https://github.com/cyyeh/large-scale-data-engineering/blob/master/cloud-computing/01-CloudComputing.pdf)
- Supplementary Materials
    - [The Datacenter as a Computer: An Introduction to the Design of Warehouse-Scale Machines](https://github.com/cyyeh/large-scale-data-engineering/blob/master/cloud-computing/datacenter-as-computer.pdf)
    - [Cloud Computing according to Wikipedia](https://en.wikipedia.org/wiki/Cloud_computing)
    - [Amazon Whitepaper: Architecting for the Cloud](https://github.com/cyyeh/large-scale-data-engineering/blob/master/cloud-computing/AWS_Cloud_Best_Practices.pdf)
    - [Strata 2011: Werner Vogels, "Data Without Limits"](https://youtu.be/oNTp5spjv0w)
    - Introduction to Amazon Web Services - How to Scale your Next Idea on AWS : A Love Story - Jinesh Varia (Updated Jan 2014)
        - [Video](https://www.youtube.com/watch?v=wgiYfUs7rNY)
        - [Slide](https://www.slideshare.net/AmazonWebServices/building-powerful-web-applications-in-the-aws-cloud-a-love-story-jinesh-varia)