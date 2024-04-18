---
title: "Monitoring NiFi - Introduction"
date: "2017-05-11"
categories: 
  - "apache-nifi"
tags: 
  - "data"
  - "flow"
  - "hdf"
  - "hortonworks"
  - "monitoring"
coverImage: "screen-shot-2016-08-13-at-8-18-54-pm.png"
---

[Apache NiFi 1.2.0 has just been released](https://nifi.apache.org/download.html) with a lot of very cool new features... and I take this opportunity to start a series of articles around monitoring. This is a recurring subject and I often hear the same questions. This series won't provide an exhaustive list of the ways you can use to monitor NiFi (with or without [HDF](https://hortonworks.com/products/data-center/hdf/)) but, at least, it should get you started!

Here is a quick summary of the subjects that will be covered:

- [Bootstrap notifier](http://pierrevillard.com/2017/05/11/monitoring-nifi-bootstrap-notifier/)
- [Logback configuration](http://pierrevillard.com/2017/05/12/monitoring-nifi-logback-configuration/)
- [SiteToSite (S2S) reporting tasks](http://pierrevillard.com/2017/05/13/monitoring-nifi-site2site-reporting-tasks/)
    - [Bulletin alerting](http://pierrevillard.com/2017/05/13/monitoring-nifi-site2site-reporting-tasks/)
    - [Disk monitoring](http://pierrevillard.com/2017/05/13/monitoring-nifi-site2site-reporting-tasks/)
    - [Memory monitoring](http://pierrevillard.com/2017/05/13/monitoring-nifi-site2site-reporting-tasks/)
    - [Back pressure alerting](http://pierrevillard.com/2017/05/13/monitoring-nifi-site2site-reporting-tasks/)
- [Workflow SLA](http://pierrevillard.com/2017/05/15/monitoring-nifi-workflow-sla/)
- [Ambari reporting task and Grafana](http://pierrevillard.com/2017/05/16/monitoring-nifi-ambari-grafana/)
- [Scripted reporting tasks](http://pierrevillard.com/2017/05/17/monitoring-nifi-scripted-reporting-task/)

For this series of article, I will use, as a demo environment, a 4-nodes HDF cluster (running on CentOS 7):

- One management node (for [Ambari](https://ambari.apache.org/), Ambari metrics, [Grafana](https://grafana.com/), etc)
- Three nodes for NiFi - [secured using the NiFi CA](http://pierrevillard.com/2016/11/29/apache-nifi-1-1-0-secured-cluster-setup/) (provided with the NiFi toolkit)

I'm using HDF to take advantage of Ambari to ease the deployment but this is not mandatory for what I'm going to discuss in the articles (except for stuff around the Ambari reporting task obviously).

I will not cover how setting up this environment but if this is something you are looking after, feel free to ask questions (here or on the [Hortonworks Community Connection](https://community.hortonworks.com/answers/index.html)) and to have a look into [Hortonworks documentation](http://docs.hortonworks.com/index.html) about HDF.

I don't want to write a single (very) long article and for the sake of clarity there is one article per listed subject. Also, I'll try to update the articles to stick as best as possible to latest features provided by NiFi over time.

Also, if you feel that some subjects should be added to the list, let me know and I'll do my best to cover other monitoring-related questions.
