# [Cloud Computing](https://hackmd.io/@mit-distributed-systems-engineering/cloud-computing)

course website: https://event.cwi.nl/lsde/2019/cloud.shtml

## Overview 

In this lecture we first define Big Data in terms of data management problems with the ==three V=='s: (i) Volume: the data is large, (2) Variety: the data is often not clean and tabular, but messy (text, even images), (3) Velocity: new data keeps arriving continuously. We also discuss ==Power Laws== in many Big Data problems: the mass of data is in the long tail, the long tail cannot be ignored as it may represent the majority of all datapoints. Power Law distributions are typical in social networks, e.g. the distribution of amount of twitter followers has a power law distribution.

We explain ==Eric Brewers CAP theorem==: a global software system cannot achieve all three of ==Consistency== (reads always reflect the latest updates), ==Availability== (the system is always up), ==Partitionability== (the system is resistant against loss of communication between datacenters), and describe concepts as replication and sharding and their consequences in terms of read speed, update speed and consistency.
As we work on large computer infrastructures, we distinguish between three related areas: (1) ==Super Computing==, where performance is king and programmability a side-issue, (2) ==Cluster Computing==, which is about quickly getting things done on large clusters of unreliable machines, and (3) ==Cloud Computing==, where computation is performed in large clusters operated by third party, sold as a commodity service to users based on their actual use, seamlessly allowing to get more or less of it based on their needs (elasticity). In the practicum where we use ==Hadoop== (cluster computing) on ==Amazon Web Services== (cloud computing) we hence combine (2) and (3). Super computing is out of scope for this module. In cloud computing, we further distinguish between ==IaaS==: Infrastructure-as-a-Service (virtual machines for rent, e.g. Amazon EC2), ==PaaS==: Platform as-a-service (database system for rent, e.g. Amazon Redshift) and ==SaaS==: Software-as-a-Service (application for rent, e.g. Microsoft Office 365 or Salesforce).

The last part of the lecture describes Amazon Web Services (AWS) in some detail to prepare you for the practicum. Services to remember are ==S3== block storage (infinitely scaling storage -- presumably implemented by spreading storage with replication over the hundreds of thousands of machines Amazon owns), ==EBFS== filesystems which are virtual disks that store their data remotely on S3 but do a lot of caching to avoid network traffic, and the ==EC2== service that allows to power up virtual machines in the cloud (and some of the options to choose from in terms of CPU, RAM, and local disks aka "ephimeral storage" which are empty when you start-up).

## Course Materials

- [Course Introduction](https://github.com/cyyeh/large-scale-data-engineering/blob/master/cloud-computing/00-Intro.pdf)
- [Cloud Computing](https://github.com/cyyeh/large-scale-data-engineering/blob/master/cloud-computing/01-CloudComputing.pdf)

## References

- [The Datacenter as a Computer: An Introduction to the Design of Warehouse-Scale Machines](https://github.com/cyyeh/large-scale-data-engineering/blob/master/cloud-computing/datacenter-as-computer.pdf)
- [Cloud Computing according to Wikipedia](https://en.wikipedia.org/wiki/Cloud_computing)
- [Amazon Whitepaper: Architecting for the Cloud](https://github.com/cyyeh/large-scale-data-engineering/blob/master/cloud-computing/AWS_Cloud_Best_Practices.pdf)
- [Strata 2011: Werner Vogels, "Data Without Limits"](https://youtu.be/oNTp5spjv0w)
- [Introduction to Amazon Web Services - How to Scale your Next Idea on AWS : A Love Story - Jinesh Varia (Updated Jan 2014)](https://www.slideshare.net/AmazonWebServices/building-powerful-web-applications-in-the-aws-cloud-a-love-story-jinesh-varia)