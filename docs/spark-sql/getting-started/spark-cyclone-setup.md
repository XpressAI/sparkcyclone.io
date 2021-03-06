---
sidebar_position: 3
---
# Spark Cyclone Installation Guide


Before proceeding with the installation, ensure that you have completed the Hadoop and Spark setup for the Vector Engine.

## 1. Update Hadoop YARN VE Resource Allocation

Depending on the number of VE and RAM available, please adjust the numbers in yarn-site.xml accordingly. The following configurations was for 2 VEs.

	$ vi /opt/hadoop/etc/hadoop/yarn-site.xml

	<property>
        <name>yarn.nodemanager.resource.memory-mb</name>
        <value>104448</value>
    </property> 
    <property>
        <name>yarn.scheduler.maximum-allocation-mb</name>
        <value>13056</value>
    </property>
    <property>
        <name>yarn.nodemanager.resource.cpu-vcores</name>
        <value>24</value>
    </property>
    <property>
        <name>yarn.scheduler.maximum-allocation-vcores</name>
        <value>24</value>
    </property>

## 2. Add getVEsResources.py

getVEsResources.py lists down the available VEs in order for Cyclone to efficiently delegate resources.

Please add the following script to `/opt/spark/`.


```
#!/usr/bin/env python

import subprocess

lines = subprocess.check_output(['/opt/nec/ve/bin/ps', 'ax']).split('\n')
ves = []
current_ve = None
for line in lines:
    if line.startswith("VE Node:"):
        ve_id = int(line.split(': ')[1])
        current_ve = {
            'id': ve_id,
            'procs': []
        }
        ves.append(current_ve)
    elif line.strip().startswith("PID TTY"):
        pass
    elif len(line.strip()) == 0:
        pass
    else:
        parts = line.split()
        proc = {
            'pid': parts[0],
            'tty': parts[1],
            'state': parts[2],
            'time': parts[3],
            'command': parts[4]
        }
        current_ve['procs'].append(proc)


ves.sort(key=lambda x: len(x['procs']) == 8)

ids = ",".join(['"' + str(x['id']) + '"' for x in ves])
print('{"name": "ve", "addresses": [' + ids + ']}')
```

## 3. Check Hadoop Status

If you are running the job from another user such as root, ensure that the user has been added from user hadoop.

    # cd /opt/hadoop/
	$ bin/hdfs dfs -mkdir /user/<otheruser>
	$ bin/hdfs dfs -chown <otheruser> /user/<otheruser>

Start Hadoop

    $ sbin/start-dfs.sh
    $ sbin/start-yarn.sh

Open Hadoop YARN Web UI to verify that the settings are updated. 

    # if you are SSHing into the server from a remote device, don't forget to forward your port.
    $ ssh root@serveraddress -L 8088:localhost:8088 

As seen from the ```Cluster Nodes``` tab, the Memory Total, VCores Total, as well as Maximum Allocation is updated.

![image](https://user-images.githubusercontent.com/68586800/137414646-4ce66a4e-2f4f-4817-a5a1-686ab349a2a3.png)


## 4. Installation

Spark Cyclone was designed to be easily swappable for normal Spark. You would need to download the latest jar release from https://github.com/XpressAI/SparkCyclone/releases, then drop it to `/opt/cyclone/${USER}/`.
To use it, simply add spark-cyclone-sql-plugin.jar to your jar can executor config: 
```
$SPARK_HOME/bin/spark-submit \
    --master yarn \
    --num-executors=8 --executor-cores=1 --executor-memory=7G \
    --name job \
    --jars /opt/cyclone/${USER}/spark-cyclone-sql-plugin.jar \
    --conf spark.executor.extraClassPath=/opt/cyclone/${USER}/spark-cyclone-sql-plugin.jar \
    --conf spark.plugins=com.nec.spark.AuroraSqlPlugin \
    --conf spark.executor.resource.ve.amount=1 \
    --conf spark.executor.resource.ve.discoveryScript=/opt/spark/getVEsResources.py \
    job.py
```
Please refer to the [configuration](https://www.sparkcyclone.io/docs/spark-sql/configuration/spark-cyclone-configuration) page for more details.


## 5. Building
Follow the steps if you would like to build Spark Cyclone.

### Build Tools

Ensure that you have both java and javac installed. You also need sbt, java-devel, as well as devtoolset-11.

	$ yum install centos-release-scl-rh    	
	$ yum install devtoolset-11-toolchain
	
	$ curl -L https://www.scala-sbt.org/sbt-rpm.repo > sbt-rpm.repo
	$ mv sbt-rpm.repo /etc/yum.repos.d/
	$ yum install sbt
	$ yum install java-devel
    

### Clone Spark Cyclone repo and build

	$ scl enable devtoolset-11 bash
	$ git clone https://github.com/XpressAI/SparkCyclone
	$ cd SparkCyclone
	$ sbt assembly

The `sbt assembly` command will compile the spark-cyclone-sql-plugin.jar file at:
    target/scala-2.12/spark-cyclone-sql-plugin-assembly-0.1.0-SNAPSHOT.jar

## 6. Installing into to cluster machines

The `deploy local` command will compile and copy the spark-cyclone-sql-plugin.jar into /opt/cyclone.  If you have a 
cluster of machines you can copy the jar file to them by running the deploy command with the hostname.

```bash

for ssh_host in `cat hadoop_hosts`
do
  sbt "deploy $ssh_host"
done
```
