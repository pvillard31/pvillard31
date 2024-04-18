---
title: "US presidential election via Twitter using Apache NiFi, Spark, Hive and Zeppelin"
date: "2016-04-29"
categories: 
  - "apache-nifi"
tags: 
  - "apache-hive"
  - "apache-spark"
  - "apache-zeppelin"
  - "data"
  - "etl"
  - "flow"
  - "hdp"
  - "hortonworks"
  - "processing"
  - "streaming"
  - "twitter"
coverImage: "sentimentpresfeature.png"
---

This article describes a frequency and sentiment analysis based on real-time tweets streams in relation to the four main candidates in the US Presidential Election.

The main objective was to deploy and to test the available connector between Apache NiFi and Apache Spark, so I decided to implement the following use case:

- Using [Apache NiFi](https://nifi.apache.org/), get filtered tweets related to US presidential election
- Using [Apache Spark](http://spark.apache.org/), get the stream of tweets from Apache NiFi
- Use [Spark streaming](http://spark.apache.org/streaming/) to transform and store the data into an [Apache Hive](https://hive.apache.org/) table
- Use [Apache Zeppelin](https://zeppelin.incubator.apache.org/) notebook to display real-time results

At the end, I get real time analytics such as:

- frequency of tweets along the time per candidate
- percentage of negative, positive and neutral tweets per candidate
- opinion trends along the time for each candidate

The article is available on [Hortonworks Community Connection website](https://community.hortonworks.com/articles/30213/us-presidential-election-tweet-analysis-using-hdfn.html). And as always, please feel free to comment and/or ask questions.
