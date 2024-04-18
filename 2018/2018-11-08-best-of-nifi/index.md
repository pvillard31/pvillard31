---
title: "Best of NiFi"
date: "2018-11-08"
coverImage: "screen-shot-2016-08-13-at-8-18-54-pm.png"
---

A lot of very good stuff about NiFi is all over the internet: a lot of posts, videos, resources shared by community members and users of this great project. This page is a genuine effort to link the best resources in case you want to improve your knowledge about NiFi or learn about techniques and/or interesting use cases.

**Note 1** - Bear with me, I'm just starting to gather the best resources and your help is truly appreciated in case you think some resources should be added to the list. To do so, feel free to comment below. I'll do my best to keep things up to date.

**Note 2** - If you're looking for somewhere to start, **best is to start with the [official documentation](https://nifi.apache.org/docs/nifi-docs/)** and, if you're curious, the [Confluence pages](https://cwiki.apache.org/confluence/display/NIFI). Also, I recommend you to subscribe to the [mailing lists](https://nifi.apache.org/mailing_lists.html) and to join us on [Slack](https://join.slack.com/t/apachenifi/shared_invite/enQtNDI2NDMyMTY3MTA5LWJmZDI3MmM1ZmYyODQwZDYwM2MyMDY5ZjkyMDkxY2JmOGMyNmEzYTE0MTRkZTYwYzZlYTJkY2JhZTYyMzcyZGI).

**Note 3** - I'll try to keep things as up to date as possible but things are moving really quick in this project and I cannot guarantee that all the technical content of the below posts is still valid (but it should be!).

* * *

## NiFi Framework

- [Content repository archiving - how does it work?](https://community.hortonworks.com/articles/82308/understanding-how-nifis-content-repository-archivi.html)
- [Dissecting a NiFi connection](https://community.hortonworks.com/articles/184990/dissecting-the-nifi-connection-heap-usage-and-perf.html)
- [Load-balancing with Site-to-Site](https://community.hortonworks.com/articles/109629/how-to-achieve-better-load-balancing-using-nifis-s.html)
- [Load-balanced connections](https://blogs.apache.org/nifi/entry/load-balancing-across-the-cluster) (+ [here](https://pierrevillard.com/2018/10/29/nifi-1-8-revolutionizing-the-list-fetch-pattern-and-more/))
- [Component versioning](https://bryanbende.com/development/2017/05/10/apache-nifi-1-2-0-component-versioning)
- [Component class loading](https://bryanbende.com/development/2016/11/24/apache-nifi-class-loading)
- [Process group variables](https://community.hortonworks.com/articles/155823/introduction-into-process-group-variables.html)

## NiFi installation and deployment

- [Best practices for High Perf NiFi installation](https://community.hortonworks.com/articles/7882/hdfnifi-best-practices-for-setting-up-a-high-perfo.html)
- [Load balancer in front of NiFi](http://ijokarumawak.github.io/nifi/2016/11/01/nifi-cluster-lb/) (+ [here](https://pierrevillard.com/2017/02/10/haproxy-load-balancing-in-front-of-apache-nifi/))
- [Deploying NiFi on Google Cloud with Terraform](http://pierrevillard.com/2019/08/21/nifi-with-oidc-using-terraform-on-the-google-cloud-platform/)

## NiFi Security

- [Encrypted provenance repository](https://alopresto.github.io/ewapr/)
- [Securing keytab access](https://bryanbende.com/development/2018/04/09/apache-nifi-secure-keytab-access)
- [Ranger policies for NiFi](https://community.hortonworks.com/articles/115770/nifi-ranger-based-policy-descriptions.html)
- [Authentication with OpenID Connect](https://bryanbende.com/development/2017/10/03/apache-nifi-openid-connect)
- [Secured NiFi cluster](https://bryanbende.com/development/2018/10/23/apache-nifi-secure-cluster-setup)  (+ [here](https://pierrevillard.com/2016/11/29/apache-nifi-1-1-0-secured-cluster-setup/)) (+ [here](https://bryanbende.com/development/2016/08/17/apache-nifi-1-0-0-authorization-and-multi-tenancy))
- [Secured Site-to-Site](https://bryanbende.com/development/2016/08/30/apache-nifi-1.0.0-secure-site-to-site)
- [Authorizations with LDAP synchronization](https://pierrevillard.com/2017/12/22/authorizations-with-ldap-synchronization-in-apache-nifi-1-4/)

## Record API in NiFi

- [Record oriented data in NiFi](https://blogs.apache.org/nifi/entry/record-oriented-data-with-nifi)
- [Records and Schema registries](https://bryanbende.com/development/2017/06/20/apache-nifi-records-and-schema-registries)
- [Real-time SQL on Event Streams](https://blogs.apache.org/nifi/entry/real-time-sql-on-event)

## NiFi Monitoring

- [General guidelines about monitoring](https://pierrevillard.com/2017/05/11/monitoring-nifi-introduction/)
- [Monitoring Driven Development](https://pierrevillard.com/2018/08/29/monitoring-driven-development-with-nifi-1-7/) (+ [here](https://pierrevillard.com/2018/02/07/fod-paris-jan-18-nifi-registry-and-workflow-monitoring-with-a-use-case/))

## "How-to" about processors

- [List/Fetch pattern in NiFi](https://pierrevillard.com/2017/02/23/listfetch-pattern-and-remote-process-group-in-apache-nifi/)
- [Integrating NiFi and Kafka](http://bryanbende.com/development/2016/09/15/apache-nifi-and-apache-kafka)
- [Incremental Fetch with QueryDatabaseTable](https://community.hortonworks.com/articles/51902/incremental-fetch-in-nifi-with-querydatabasetable.html)
- [Wait/Notify pattern in NiFi](http://ijokarumawak.github.io/nifi/2017/02/02/nifi-notify-batch/) (+ [here](https://pierrevillard.com/2018/06/27/nifi-workflow-monitoring-wait-notify-pattern-with-split-and-merge/))
- [3 part series about MySQL CDC in NiFi](https://community.hortonworks.com/articles/113941/change-data-capture-cdc-with-apache-nifi-version-1-1.html)
- [XML Processing](https://pierrevillard.com/2018/06/28/nifi-1-7-xml-reader-writer-and-forkrecord-processor/) (+ [here](https://pierrevillard.com/2017/09/07/xml-data-processing-with-apache-nifi/))
- [JSON transformations with Jolt](http://ijokarumawak.github.io/nifi/2016/11/22/nifi-jolt/)
- [Processing multi-lines logs](https://bryanbende.com/development/2017/10/04/apache-nifi-processing-multiline-logs)

## Scripted components

- [ExecuteScript cookbook part 1](https://community.hortonworks.com/articles/75032/executescript-cookbook-part-1.html)
- [ExecuteScript cookbook part 2](https://community.hortonworks.com/content/kbentry/75545/executescript-cookbook-part-2.html)
- [ExecuteScript cookbook part 3](https://community.hortonworks.com/articles/77739/executescript-cookbook-part-3.html)

## MiNiFi and C2 server

- [IIoT system using NiFi, MiNiFi, C2 Server, MQTT and Raspberry Pi](https://medium.freecodecamp.org/building-an-iiot-system-using-apache-nifi-mqtt-and-raspberry-pi-ce1d6ed565bc)
- [Running visual quality inspection at the edge using Google Cloud and Apache NiFi & MiNiFi](http://pierrevillard.com/2019/10/29/running-visual-quality-inspection-at-the-edge-with-google-cloud-and-apache-nifi-minifi/)

## NiFi Registry

- [Flow Development Life Cycle with NiFi Registry](https://pierrevillard.com/2018/04/09/automate-workflow-deployment-in-apache-nifi-with-the-nifi-registry/)
- [Full video demo of SDLC with NiFi and NiFi Registry](https://www.youtube.com/watch?v=yKmVBTeZS4c)
- [Git backend and event hooks in NiFi Registry](https://bryanbende.com/development/2018/06/20/apache-nifi-registry-0-2-0)
- [Git backend in NiFi Registry](https://community.hortonworks.com/articles/210286/storing-apache-nifi-versioned-flows-in-a-git-repos.html)

## NiFi Tools

- [NiPy API - a Python client for NiFi and NiFi Registry](https://github.com/Chaffelson/nipyapi)
- [NiFi on Docker](https://hub.docker.com/r/apache/nifi/)

## Use cases, fun with NiFi, and others

- [Lots of nice articles around IoT/AI/ML + (Mi)NiFi by Tim Spann](https://dzone.com/users/297029/bunkertor.html)
- [Best talks about NiFi at DataWorks Summits](https://www.youtube.com/user/HadoopSummit/search?query=nifi)

###### (last update 08/11/2018)
