---
title: "Hue/Oozie causing CPU overload"
date: "2018-07-30"
categories: 
  - "hortonworks"
tags: 
  - "administration"
  - "hue"
  - "oozie"
  - "security"
---

Quick post about an issue I faced today on one of the clusters: I received an alert about abnormal high CPU use on one of the master nodes. A quick htop gave me the culprit: the Oozie server hosted on this node.

I looked at the logs and didn't see anything unusual in the oozie.log file. But by looking at the oozie-audit.log file, I noticed a very large number of requests being issued by Hue and proxifying users:

```
# sed 's/.* DoAs user \[\(.*\)\] Request .*/\1/g' oozie-audit.log | sort | uniq -c | sort -nr | head
279616               jdoe
27902                zoaks
16018                mparisien
14025                gkass
12211                lzastrow
9730                 sleaf
7460                 sladwig
6048                 vespinoza
5815                 lkonen
2862                 lrayburn
```

It appeared my John Doe was issuing more than 5 requests per second to the Oozie server using Hue causing the high CPU consumption.

When being on the Oozie dashboards in Hue with your browser, there is an auto-refresh feature issuing requests to the Oozie server every 5 seconds to get the latest statuses. Problem is if a user is opening multiple tabs in the browser, it can lead to a lot of requests. Now... if the user forgets to close the browser and remains connected, you have a nice DDoS-like situation.

By looking at the Hue documentation, I thought I found a solution with the below:

```
[desktop]
[[auth]]
# Users will automatically be logged out after ‘n’ seconds of inactivity.
# A negative number means that idle sessions will not be timed out.
idle_session_timeout=-1
```

I tried setting this value to 600 seconds (10 minutes) to get inactive people automatically logged out. It works fine when you're staying on a static page in Hue but not if you're staying on the Oozie pages... the auto-refresh is keeping you "active" even though you're not.

The only option I found is to use the ttl (time-to-live) parameter to define when the cookie will expire and force the user to authenticate again. The issue with this parameter is that it'll log out the user even though the user is active and actually using Hue.

To avoid any unpleasant user experience, you can set this parameter to something like 28800 (8 hours):

```
[desktop]
[[auth]]
ttl=28800
```

It does not solve the original issue because you'll keep your Oozie server receiving a lot of requests for 8 hours but, at least, you limit how long this situation can last.

The best solution, assuming you have installed multiple Oozie instances for high availability behind a load balancer, is to configure the LB to extract the user name from the requested URL (&doAs=<user>) and to throttle the number of requests issued by a single user. That will provide the best protection without impacting the user experience. Look at your LB's documentation to configure such a solution.
