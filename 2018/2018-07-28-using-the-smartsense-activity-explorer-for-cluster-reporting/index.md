---
title: "Using the SmartSense Activity Explorer for cluster reporting"
date: "2018-07-28"
tags: 
  - "administration"
  - "apache-hive"
  - "apache-ranger"
  - "hortonworks"
  - "jdbc"
  - "security"
  - "smartsense"
coverImage: "hortonworks.jpg"
---

In the [Hortonworks Data Platform](https://docs.hortonworks.com/), there is [SmartSense](https://docs.hortonworks.com/HDPDocuments/SS1/SmartSense-1.5.0/introduction/content/ss_smartsense_intro.html), a service that analyzes cluster diagnostic data, identifies potential issues, and recommends specific solutions and actions.

SmartSense is made of multiple components and one of the component is the [Activity Explorer](https://docs.hortonworks.com/HDPDocuments/SS1/SmartSense-1.5.0/introduction/content/ss_activity_analysis.html) which is a customized [Zeppelin notebook](https://zeppelin.apache.org/) used to access and display the data collected by the Activity Analyzer instances stored in an [HBase](https://hbase.apache.org/) instance and accessed using [Phoenix](https://phoenix.apache.org/).

The Activity Explorer gives access to a lot of very useful data when administrating a cluster. For an exhaustive list, have a look to [the documentation here](https://docs.hortonworks.com/HDPDocuments/SS1/SmartSense-1.5.0/using/content/ss_accessing_the_activity_explorer.html).

By default, this Activity Explorer / Zeppelin is configured with the Phoenix interpreter only. The idea of this post is to describe how we can add the JDBC interpreter (or any other interpreter) to allow administration teams using this specific Zeppelin instance as a more general tool for cluster reporting.

One might wonder why I'm not using the Zeppelin service available in the HDP stack. The reason is quite simple: usually, the Zeppelin instances would be deployed on the edge nodes (to be used by the project teams / users of the cluster) while the Activity Explorer would be deployed on an administration node and only accessed by the administrators of the cluster. The idea is to keep Zeppelin instances separated based on the purpose.

First step is to package the JDBC interpreter. Go on the node where you installed the Zeppelin service (where all the interpreters are installed) - not the node where you installed the Activity Explorer component.

```
cd /usr/hdp/current/zeppelin-server/interpreter/
zip -r jdbc.zip jdbc/
```

And deploy this ZIP file on the node where is installed the Activity Explorer:

```
cd /usr/hdp/share/hst/activity-explorer/interpreter/
unzip jdbc.zip
```

Restart the Activity Explorer component so that the interpreter is available for configuration.

Go to the interpreter configuration page and add a new one, selecting the JDBC type. Configure the interpreter as needed based on your cluster (you can check the configuration you set for this interpreter in the Zeppelin service). In particular, you'll need:

```
zeppelin.jdbc.auth.type=KERBEROS
zeppelin.jdbc.principal=<principal of the activity explorer>
zeppelin.jdbc.keytab.location=<keytab of the activity explorer>
hive.proxy.user.property=hive.server2.proxy.user
```

_Note_: do not use \_HOST in the principal name, use the host FQDN instead.

I also **strongly** recommend you to configure SSL on the Activity Explorer as well as configuring proper authentication/authorization mechanisms. You can do all that through Ambari as you'd do for the Zeppelin service (have a look at [the documentation here](https://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.6.5/bk_zeppelin-component-guide/content/ch_auth.html)).

Since the Activity Explorer account is going to proxy your requests to Hive through the JDBC interpreter, you need to add the proper proxy rules:

```
hadoop.proxyuser.activity_explorer.groups=<administrator group>
hadoop.proxyuser.activity_explorer.hosts=<activity explorer host>
```

And you'll have to restart the appropriate services.

If you stop here and restart the Activity Explorer component, you'll loose your JDBC interpreter configuration because all of the interpreter configuration of the Activity Explorer is managed by Ambari and reset at each component restart. To prevent the loss of your configuration, you need to copy the content of the file:

```
/etc/smartsense-activity/conf/interpreter.json
```

(content of this file has been updated by the Activity Explorer after you added the JDBC interpreter)

And paste this content in Ambari / SmartSense / Advanced / Advanced activity-zeppelin-interpreter. This way, your configuration will remain the same.

_Note_: keep in mind that all this procedure might have to be done again after a SmartSense upgrade since it's not the default deployment.

You're now all set! If you're wondering what can be done with the JDBC interpreter to enhance the cluster administration tasks... the first thing I can recommend is to create Hive tables on top of the Ranger audits stored in HDFS so that you can create long term reports based on all the cluster audits (if you're using Solr for the Ranger audits, this data is only stored for a short period of time, default is 90 days). Creating Hive tables on top of the data sitting in HDFS can really be useful if you have compliance/security teams looking for audits reporting.

You could also use the JDBC interpreter to directly access the data in the database backend used for some services like Ambari, Ranger, Hive, etc. It can provide interesting data to build useful reports.

As always, thanks for reading, and feel free to ask questions / leave a comment.
