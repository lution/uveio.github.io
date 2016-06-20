---
layout: post
title:  "UVE Data Flow and Monitoring"
author: <a href="https://github.com/fwso">James Tang</a>
date:   2016-06-20 04:01:01
categories:
  - Architecture
tags:
  - "Big Data"
  - "Data Analysis"
---

## Data Sources

[Weibo](!http://weibo.com) UVE Team serves billions requests everyday, consequently,
we have to process billions data everyday.

The primary data sources include:

- Requests log
- Response log
- ADs impression log
- ADs actions(click, like, follow, etc)
- Internal service logs(ADs Bid, ADs rendering, etc)

## Data Flow Overview

The following diagram shows the overall data flow of UVE.

![UVE Data Flow Diagram](/images/UVEDataFlow.png)

## Data Transfer

As the diagram shows, many data sources are log files. In order process the logs
in realtime, we developed [YYPush](https://github.com/uveio/YYPush) to transfer
log file to [Apache Kafka](http://kafka.apache.org/).

[Gobblin](https://github.com/linkedin/gobblin) is used to persist data onto HDFS,
on which offline processing runs.

Realtime processing is implemented with [Apache Samza](http://samza.apache.org/) and
[Druid](http://druid.io/).

## Data Storage

HDFS is our primary data storage, but due to some reason, the [Apache Cassandra](http://cassandra.apache.org/) is used as the [deep storage](http://druid.io/docs/0.9.0/dependencies/deep-storage.html) of Druid.

## Data Analysis

### Realtime

Thanks to [Druid](http://druid.io/), realtime data analysis become much straightforward and more flexible.

### Offline

Beside realtime analysis, we have to do some more complex offline processing and analysis.
Most the offline processing is based on Hadoop MapReduce, Hive, HBase, etc.

## Data Monitoring

Monitoring is also very important. We have to discover what's going wrong when a new feature is released in realtime, and we want to be notified when some accident occurred or the impression is not as expected.

[Graphite](https://github.com/graphite-project/) makes our work much easier for realtime
monitoring. Based on Graphite, we could monitor our data horizontally and vertically easily. The horizontal means compare current data with historical data, and vertical means compare data from different sources at the same time point, as the following diagram shows.

![UVE Horizontal/Vertical Data Monitoring](/images/uve-horizontal-vertical-data-monitoring.png)

There are many monitoring tools that work with Graphite, such as [Graphite-beacon](https://github.com/klen/graphite-beacon), [Cabot](https://github.com/arachnys/cabot), for more information, see [Tools That work with Graphite](http://graphite.readthedocs.io/en/latest/tools.html)


