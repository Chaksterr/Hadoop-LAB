# Hadoop Cluster - Automated Setup

Automated Hadoop cluster with HDFS, YARN, MapReduce, and Hive.

## Architecture

```
Master Node:          DataNode 1:        DataNode 2:
- NameNode           - DataNode          - DataNode
- ResourceManager    - NodeManager       - NodeManager
- JobHistoryServer
- Hive + MySQL
```

## Installation

```bash
vagrant up
```

Wait 30-45 minutes. Everything installs automatically.

## Start/Stop

**Stop:**
```bash
ansible-playbook -i ansible/inventory.ini ansible/stop_cluster.yml
vagrant halt
```

**Start:**
```bash
vagrant up
ansible-playbook -i ansible/inventory.ini ansible/start_cluster.yml
```

## Access

**Web UIs:**
- HDFS: http://localhost:9870
- YARN: http://localhost:8088
- Job History: http://localhost:19888
- Hive: http://localhost:10002

**SSH:**
```bash
vagrant ssh master
sudo su - hadoop
```

## Usage

**HDFS:**
```bash
vagrant ssh master
sudo su - hadoop
export HADOOP_HOME=/home/hadoop/hadoop

$HADOOP_HOME/bin/hdfs dfs -ls /
$HADOOP_HOME/bin/hdfs dfs -put file.txt /
```

**Hive:**
```bash
vagrant ssh master
sudo su - hadoop
export HADOOP_HOME=/home/hadoop/hadoop
export HIVE_HOME=/home/hadoop/hive

$HIVE_HOME/bin/hive -e "SHOW DATABASES;"
```

## Verify

```bash
./verify.sh
```

## Clean Reinstall

```bash
vagrant destroy -f
vagrant up
```

