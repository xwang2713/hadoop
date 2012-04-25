Overview
--------

The Apache Hadoop software library is a framework that allows for the
distributed processing of large data sets across clusters of computers
using a simple programming model.

It is designed to scale up from single servers to thousands of machines,
each offering local computation and storage. Rather than rely on hardware
to deliver high-avaiability, the library itself is designed to detect
and handle failures at the application layer, so delivering a
highly-availabile service on top of a cluster of computers, each of
which may be prone to failures.

Hadoop consists of the following two core components:

* Hadoop Distributed File System (HDFS™) is the primary storage system
  used by Hadoop applications. HDFS creates multiple replicas of data
  blocks and distributes them on compute nodes throughout a cluster to
  enable reliable, extremely rapid computations.

* Hadoop MapReduce is a programming model and software framework for
  writing applications that rapidly process vast amounts of data in
  parallel on large clusters of compute nodes.

Usage
-----

This charm supports the following Hadoop roles:

* HDFS: namenode, secondarynamenode and datanode
* MapReduce: jobtracker, tasktracker

This supports deployments of Hadoop in a number of configurations.

Combined HDFS and MapReduce
+++++++++++++++++++++++++++

In this configuration, the MapReduce jobtracker is deployed on the same
service units as HDFS namenode and the HDFS datanodes also run MapReduce
tasktrackers::

    juju deploy hadoop hadoop-master
    juju deploy hadoop hadoop-slavecluster
    juju add-unit -n 2 hadoop-slavecluster
    juju add-relation hadoop-master:namenode hadoop-slavecluster:datanode
    juju add-relation hadoop-master:jobtracker hadoop-slavecluster:tasktracker

Separate HDFS and MapReduce
+++++++++++++++++++++++++++

In this configuration the HDFS and MapReduce deployments operate on
different service units as separate services::

    juju deploy hadoop hdfs-namenode
    juju deploy hadoop hdfs-datacluster
    juju add-unit -n 2 hdfs-datacluster
    juju add-relation hdfs-namenode:namenode hdfs-datacluster:datanode

    juju deploy hadoop mapred-jobtracker
    juju deploy hadoop mapred-taskcluster
    juju add-unit -n 2 mapred-taskcluster
    juju add-relation mapred-jobtracker:mapred-namenode hdfs-namenode:namenode
    juju add-relation mapred-taskcluster:mapred-namenode hdfs-namenode:namenode    
    juju add-relation mapred-jobtracker:jobtracker mapred-taskcluster:tasktracker

In the long term juju should support improved placement of services to
better support this type of deployment.  This would allow mapreduce services
to be deployed onto machines with more processing power and hdfs services
to be deployed onto machines with larger storage.

HDFS with HBase
+++++++++++++++

This charm also supports deployment of HBase; HBase requires that append mode
is enabled in DFS - this can be set by providing a config.yaml file::

    hdfs-namenode:
        hbase: true
    hdfs-datacluster:
        hbase: true

Its really important to ensure that both the master and the slave services have
the same configuration in this deployment scenario.

The charm can then be use to deploy services with this configuration::

    juju deploy --config config.yaml hadoop hdfs-namenode
    juju deploy --config config.yaml hadoop hdfs-datacluster
    juju add-unit -n 2 hdfs-datacluster
    juju add-relation hdfs-namenode:namenode hdfs-datacluster:datanode

You can then associate a hdfs service deployment with a hbase service deployment::

    juju add-relation hdfs-namenode:namenode hbase-master:namenode
    juju add-relation hdfs-namenode:namenode hbase-regioncluster:namenode
    juju add-relation hdfs-namenode:namenode hbase-datacluster:namenode

See the hbase charm for more details on deploying HBase.

Words of Caution
----------------

Note that removing the relation between namenode and datanode is destructive!
The role of the service is determined at the point that the relation is added
(it must be qualified) and CANNOT be changed later!

A single hdfs-master can support multiple slave service deployments::

    juju deploy hadoop hdfs-datacluster-02
    juju add-unit -n 2 hdfs-datacluster-02
    juju add-relation hdfs-namenode:namenode hdfs-datacluster-02:datanode

This could potentially be used to perform charm upgrades on datanodes in
sets::

    juju upgrade-charm hdfs-datacluster
    (go and make some tea whilst monitoring juju debug-log)
    juju upgrade-charm hdfs-datacluster-02

Could be helpful to avoid outages (to be proven).

