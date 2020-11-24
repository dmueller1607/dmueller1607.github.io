---
layout: post
title: Fighting against random Livy session timeouts
author: Daniel Müller
date: '2020-11-24'
category: guides
tags: hadoop
summary: Fighting against random Livy session timeouts
thumbnail: post4_livy-session-timeouts/logo.png
---

# Fighting against random Livy session timeouts

Logo Information: Icon (emoji) made by [Vectors Market](https://www.flaticon.com/authors/vectors-market) from [www.flaticon.com](https://www.flaticon.com/).

In my company we set up a Hadoop cluster, containing these (and some other) components:

* Cloudera CDH 6.3
  * Apache Hadoop 3.0 (YARN, HDFS, MapReduce)
  * Apache Spark 2.4
* Apache Livy 0.7.0 (manual installation, not included in CDH)
  * We used following installation guide: http://idata.co.il/2019/10/apache-livy-a-rest-gateway-for-spark/
* Kerberos (authentication service)

One of the use cases is to use Livy for accessing the Spark resources from outside the cluster itself (e.g. an edge node for some data science stuff). Therefore, Apache Livy is a perfect solution, since it offers a REST-ful API for creating and managing interactive Spark sessions.

However, we had the problem, that when we started a Livy session, it was killed after a few minutes (random), independent if the current Spark session was in use or not at this time.

So, first of all was to look into the Livy logs, and we saw following strange output in the `livy-<user>-server.out` file:

``````
INFO sessions.InteractiveSessionManager: Deleting InteractiveSession 0 because it was inactive for more than 3600000.0 ms.
``````

That looked strange to me. The session is **not** killed after 3.600.000 ms (=1h), it is killed after a few minutes! After checking all timezones of the servers (they were all correct), I was sure that this timeout value is not correct. So, let me check, where this message is generated. Therefore I had a look into Livy's GitHub file at https://github.com/apache/incubator-livy/blob/branch-0.7/server/src/main/scala/org/apache/livy/sessions/SessionManager.scala. There's this method defined:

``````scala
def collectGarbage(): Future[Iterable[Unit]] = {
    def expired(session: Session): Boolean = {
      session.state match {
        case s: FinishedSessionState =>
          val currentTime = System.nanoTime()
          currentTime - s.time > sessionStateRetainedInSec
        case _ =>
          if (!sessionTimeoutCheck) {
            false
          } else if (session.state == SessionState.Busy && sessionTimeoutCheckSkipBusy) {
            false
          } else if (session.isInstanceOf[BatchSession]) {
            false
          } else {
            val currentTime = System.nanoTime()
            currentTime - session.lastActivity > sessionTimeout
          }
      }
    }

    Future.sequence(all().filter(expired).map { s =>
      info(s"Deleting $s because it was inactive for more than ${sessionTimeout / 1e6} ms.")
      delete(s)
    })
  }
``````

So, that method returns `true`, when

* a **session really times out** (we could exclude this case, since our server times are identical and the session is stopped after a few minutes) or 
* when the **session state is finished**, independent of any timeout value. 

So, this log message containing the timeout value can be misleading (especially for our case, where no timeout is reached, but the session state seems to go to finished state).

So, having another look into the log file tells us this information, before the deletion happens:

``````
[...]
INFO ipc.Client: Retrying connect to server: 0.0.0.0/0.0.0.0:8032. Already tried 8 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)

INFO ipc.Client: Retrying connect to server: 0.0.0.0/0.0.0.0:8032. Already tried 9 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)

[...]
``````

Livy seems to try to connect to the local machine's port 8082, but can't get there. So, we checked which Livy or Hadoop service is running on this port, and we found out (using this link: https://docs.cloudera.com/documentation/enterprise/latest/topics/cdh_ports.html), that port 8032 is used for YARN's Resource Manager. Problem here: **We run the Livy server on one of our data nodes, so it cannot reach the Resource Manager locally, since it runs on another Hadoop machine.**

Second step: Checking the Livy configs `livy.conf`:

``````python
# What spark master Livy sessions should use.
livy.spark.master = yarn

# What spark deploy mode Livy sessions should use.
livy.spark.deploy-mode = cluster

# If livy should impersonate the requesting users when creating a new session.
livy.impersonation.enabled = true

# Authentication support for Livy server
# Livy has a built-in SPnego authentication support for HTTP requests  with below configurations.
livy.server.auth.type = kerberos
livy.server.auth.kerberos.principal = HTTP/fqdn.server.de@REALM.DE
livy.server.auth.kerberos.keytab = /Livy/hdfs.keytab
livy.server.launch.kerberos.keytab /Livy/livy.keytab
livy.server.launch.kerberos.principal livy@REALM.DE
``````

Nothing conspicuous here... So in the next step we checked the environment variables, that we had to set during Livy installation:

``````
export HADOOP_CONF_DIR=/etc/hadoop/conf.cloudera.hdfs
export SPARK_HOME=/opt/cloudera/parcels/CDH/lib/spark
export LIVY_HOME=/Livy/apache-livy-0.7.0-incubating-bin
export PATH=$PATH:$LIVY_HOME/bin
``````

Okay, so Spark and Livy directories are correct for us (without them the Livy session wouldn't start at all). But what is this setting for `HADOOP_CONF_DIR`? When we took a look at the `/etc/hadoop/conf.cloudera.hdfs` directory, we saw that there was no `yarn.site.xml`, which contains important information about our YARN service! 

Checking the parent directory `/etc/hadoop` brought us the solution: We used the wrong path for `HADOOP_CONF_DIR`. We had to use the other sub-folder `/etc/hadoop/conf` which contains all important config files for our Hadoop services:

* core-site.xml
* hdfs-site.xml
* mapred-site.xml
* yarn-site.xml

So setting `export $HADOOP_CONF_DIR=/etc/hadoop/conf` and restarting the Livy server by `livy-server stop`, followed by a `livy-server start` eliminated the problem for us and our Livy sessions won't timeout while they're in use (they timeout after 1h of idle state).





