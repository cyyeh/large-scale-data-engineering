# Assignment 1

course website: https://event.cwi.nl/lsde/2019/practical1.shtml

## Overview

When is a Big Data problem really "big"? Certainly questions of size are relative. The social network data you have to work on here is in absolute terms not very big: 9 million persons; while Facebook has more than 100 times as many users, and the data you work on has all posts, likes and pictures stripped away, while this is where most of the data volume in social networks is. What is left over here, the "social graph" of who knows who, is just 5GB of compressed data. One purpose of this practical is to show you that many Big Data problems are not really big problems and can be solved with simple hardware, if one would have ample time to find an optimized solution.

This practical aims to let students think about the performance of computational tasks on large datasets. The social network query you must implement, requires to think about the data, its data distributions, as well as algorithms to navigate this data. An understanding of the interaction between data and algorithms is not only useful in order to solve problems that seem big with small hardware, but is also crucial to analyze and solve the problems that may occur in real "big" Big Data analysis. Such insights further help to understand why software systems for Big Data, such as Hadoop, were designed the way they are.

### Understand Access Patterns

One issue that makes data management problems potentially "big" problems is the complexity of the task. High complexity can come in the form of algorithms that are CPU intensive (which either need many CPU cycles per processed data element, or have quadratic or worse algorithmic complexity in terms of data size), and/or in the shape of "difficult" access patterns. The access pattern to a dataset can be sequential or scattered (random) in which case both temporal and physical locality will determine how expensive this access is going to be. You will find that there is a large difference, certainly when managing datasets larger than RAM, between sequential access, and random access without locality.

It is also important to think about what part of the data is actually needed, and which is not (could potentially be thrown away). In the practical you have the opportunity to build a data structure of your choice in the "reorg" phase, which may greatly reduce the difficulty in answering the queries.

### Practice Technical Skills

There is no obligatory language in which you have to program, however we recommend using C or C++. Java might also work, but it will eat much more memory - of which you have little - if you do not take care (suggestion: put everything as basic types in arrays and avoid using objects and standard hashmaps). Scripting languages like python are much slower and will not make you win this competition and might not even allow you to make the maximal benchmarking timeout. So, certain programming skills come in handy. We do provide an example C implementation, and the changes needed to turn that into a viable solution are limited.

Further, we require you to use git for version control over your software solutions in private github respositories (see more on using git).

In the lecture I will talk about a number of advanced data management techniques, such as columnar storage, vectorized execution (that easily benefits from SIMD instructions) and discuss inverted lists. This practicum provides opportunity to employ these in order to enhance the speed of your solution (and win the competition). Such advanced techniques are not required, though, to create an acceptable solution.

## Your Task

### Scenario: Birthday Cross-Selling

