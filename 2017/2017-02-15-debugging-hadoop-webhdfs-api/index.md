---
title: "Debugging Hadoop WebHDFS API"
date: "2017-02-15"
categories: 
  - "apache-hadoop"
tags: 
  - "debug"
  - "hadoop"
  - "hdfs"
  - "jira"
  - "webhdfs"
---

Last week, I found myself unable to use the WebHDFS REST API through an ETL tool. The only error message I got was:

```
HTTP 400 Bad Request
```

By looking at the following [documentation](https://hadoop.apache.org/docs/r1.0.4/webhdfs.html#HTTP+Response+Codes), I understood that there was only two options:

- IllegalArgumentException
- UnsupportedOperationException

Not really helping so I decided to make the API calls myself with curl requests... and here it was with a simple query to list files at the root folder of HDFS as user "123":

```
$ curl -i -X PUT -T test.txt "http://mynode:50075/webhdfs/v1/?op=LISTSTATUS&user.name=123"
HTTP/1.1 400 Bad Request
Content-Type: application/json; charset=utf-8
Content-Length: 209
Connection: close

{"RemoteException":{"exception":"IllegalArgumentException","javaClassName":"java.lang.IllegalArgumentException","message":"Invalid value: \"123\" does not belong to the domain ^[A-Za-z_][A-Za-z0-9._-]*[$]?$"}}
```

Conclusion is: the user name used with the query is checked against a regular expression and, if not validated, the above exception is returned. The default regular expression being:

```
^[A-Za-z_][A-Za-z0-9._-]*[$]?$
```

I was unable to use WebHDFS because my username was starting by numbers... I understand that this is kind of an edge case: it is rare to have such format in user names but still... I couldn't believe to be blocked because of this.

After a quick search, I found [HDFS-4983](https://issues.apache.org/jira/browse/HDFS-4983) that exposed a property in the HDFS configuration file (from Apache Hadoop 2.3.0) to change the default regular expression. Great! I changed the property (dfs.webhdfs.user.provider.user.pattern), restarted my HDFS service and tested my curl request. The request was successful so I restarted my ETL workflow... and... got the same error! Kind of unexpected...

Back to the basics: making curl requests... My ETL workflow was just trying to load a file into HDFS... So let's do that manually:

```
$ curl -i -X PUT "http://mynode:50070/webhdfs/v1/tmp/test.txt?op=CREATE&user.name=123"
HTTP/1.1 307 TEMPORARY_REDIRECT
Cache-Control: no-cache
Expires: Sat, 04 Feb 2017 10:19:38 GMT
Date: Sat, 04 Feb 2017 10:19:38 GMT
Pragma: no-cache
Expires: Sat, 04 Feb 2017 10:19:38 GMT
Date: Sat, 04 Feb 2017 10:19:38 GMT
Pragma: no-cache
X-FRAME-OPTIONS: SAMEORIGIN
Set-Cookie: hadoop.auth="u=123&p=123&t=simple&e=1486239578624&s=UrzCjP0SPpPKDJnSYB5BsKuQVKc="; Path=/; HttpOnly
Location: http://mynode:50075/webhdfs/v1/tmp/test.txt?op=CREATE&user.name=123&namenoderpcaddress=mynode:8020&createflag=&createparent=true&overwrite=false
Content-Type: application/octet-stream
Content-Length: 0

$ curl -i -X PUT -T test.txt "http://mynode:50075/webhdfs/v1/tmp/test.txt?op=CREATE&user.name=123&namenoderpcaddress=mynode:8020&createflag=&createparent=true&overwrite=false"
HTTP/1.1 400 Bad Request
Content-Type: application/json; charset=utf-8
Content-Length: 209
Connection: close

{"RemoteException":{"exception":"IllegalArgumentException","javaClassName":"java.lang.IllegalArgumentException","message":"Invalid value: \"123\" does not belong to the domain ^[A-Za-z_][A-Za-z0-9._-]*[$]?$"}}
```

As explained [here](https://hadoop.apache.org/docs/r1.0.4/webhdfs.html#CREATE), the first step is to make a request against the Name Node to get the address of a Data Node where the file data is to be written. Then make the second request directly to the Data Node to actually send the data (in this case it's my test.txt file). And... here is the error again!

So it seems that HDFS-4983 is only fixing read access against the Name Node web interface but not when calling the web interface running with a Data Node... I checked out the code (be careful to check out the code corresponding to the exact version you are running otherwise attaching a debugger won't be helpful, in my case I got the sources from Hortonworks repositories), and imported it in my Eclipse IDE. A quick search in the code to find occurrences of the regular expression lead me to [UserParam](https://github.com/apache/hadoop/blob/trunk/hadoop-hdfs-project/hadoop-hdfs-client/src/main/java/org/apache/hadoop/hdfs/web/resources/UserParam.java) class where I found a static import:

```
import static org.apache.hadoop.hdfs.DFSConfigKeys.DFS_WEBHDFS_USER_PATTERN_DEFAULT;
```

And:

```
private static Domain domain = new Domain(NAME, Pattern.compile(DFS_WEBHDFS_USER_PATTERN_DEFAULT));
```

Meaning that if this domain attribute is not overridden somewhere by a call, then the default regular expression will be used.  It really looks like what we are experiencing!

Just to confirm my assumption, I modified the bootstrap of the Data Node JVM to attach a debugger by adding the following parameters when the JVM is launched:

```
-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=9999
```

This way, in Eclipse, I can launch a remote debugger pointing to my Data Node on the port 9999 (ensure this port is opened and available) and define all the breakpoints I want. In this case I set a break point in the _channelRead0_ method of the [WebHdfsHandler](https://github.com/apache/hadoop/blob/trunk/hadoop-hdfs-project/hadoop-hdfs/src/main/java/org/apache/hadoop/hdfs/server/datanode/web/webhdfs/WebHdfsHandler.java) class (which is the entry point of the HTTP listener running in the Data Node). This way I confirmed that when checking Users Group Information:

```
ugi = ugiProvider.ugi();
```

The user name is checked using the default regular expression. By looking at the patch in HDFS-4983, it was easy to figure out what needed to be added in the code to have the property correctly set in the Data Node web handler. And that's how I created [HDFS-11391](https://issues.apache.org/jira/browse/HDFS-11391) and the associated pull request.

One line of code to fix this issue. Sometimes it's not that easy. Anyway, it gave me the opportunity to contribute to the Apache Hadoop project... Sweet.
