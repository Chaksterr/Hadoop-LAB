# 🐘 Hadoop Lab

A big-data lab that runs on a laptop: Hadoop clusters on virtual machines, built as **Infrastructure as Code**.

![Vagrant](https://img.shields.io/badge/Vagrant-1868F2?logo=vagrant&logoColor=white)
![Ansible](https://img.shields.io/badge/Ansible-EE0000?logo=ansible&logoColor=white)
![VirtualBox](https://img.shields.io/badge/VirtualBox-183A61?logo=virtualbox&logoColor=white)
![Ubuntu](https://img.shields.io/badge/Ubuntu_22.04-E95420?logo=ubuntu&logoColor=white)
![Hadoop](https://img.shields.io/badge/Hadoop_3.3.6-66CCFF?logo=apachehadoop&logoColor=black)
![Hive](https://img.shields.io/badge/Hive_3.1.3-FDEE21?logo=apachehive&logoColor=black)
![HBase](https://img.shields.io/badge/HBase_2.5.12-BE160C?logo=apache&logoColor=white)
![Spark](https://img.shields.io/badge/PySpark-E25A1C?logo=apachespark&logoColor=white)
![ZooKeeper](https://img.shields.io/badge/ZooKeeper-3.8-6DB33F?logo=apache&logoColor=white)
![Kafka](https://img.shields.io/badge/Kafka_3.9.2-231F20?logo=apachekafka&logoColor=white)
![Jupyter](https://img.shields.io/badge/Jupyter-F37626?logo=jupyter&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-4479A1?logo=mysql&logoColor=white)

One command, `vagrant up`, creates the virtual machines, installs Java and Hadoop, configures every
node and starts the cluster. There's no manual setup and no cloud account.

Each **branch** is a separate cluster that adds one part of the Hadoop ecosystem.

---

## 🌿 Branches

| Branch | Cluster | Nodes | Focus |
|---|---|---|---|
| [`hadoop-cluster`](../../tree/hadoop-cluster) | **Hadoop + Spark** | 3 | Base HDFS / YARN cluster with MapReduce and PySpark exercises |
| [`hadoop-ha`](../../tree/hadoop-ha) | **High Availability** | 4 | Two NameNodes with automatic failover through ZooKeeper and JournalNodes |
| [`hadoop-hive`](../../tree/hadoop-hive) | **Hive + HBase** | 3 | SQL on HDFS (Hive) and a NoSQL database on HDFS (HBase) |

```bash
git checkout <branch>      # pick a cluster
vagrant up                 # build it
```

---

## 🎯 What this project demonstrates

| Area | In this repo |
|---|---|
| **Infrastructure as Code** | VMs defined in a `Vagrantfile`; all software installed and configured by Ansible roles |
| **Distributed storage** | HDFS across datanodes with replication, so losing one node loses no data |
| **Distributed processing** | YARN scheduling MapReduce jobs (Java) and Spark jobs (PySpark) |
| **High availability** | Active and standby NameNodes, a 3-node ZooKeeper quorum, JournalNodes and ZKFC for automatic failover |
| **Data warehousing** | Hive with a MySQL metastore: SQL tables over HDFS files |
| **NoSQL** | HBase in high availability: active + backup HMaster, RegionServers on the datanodes, shared 3-node ZooKeeper quorum |
| **Streaming** | 3-node Kafka cluster (ZooKeeper mode), topic `INGTopic`, Python producer / consumer with a real-time chart in Jupyter |
| **Operations** | Start and stop playbooks that respect service order (for example HBase stops before HDFS) |

---

## 🏗️ Architecture

**`hadoop-cluster`: Hadoop + Spark**
```
master     NameNode · ResourceManager · JobHistory
datanode1  DataNode · NodeManager
datanode2  DataNode · NodeManager
```

**`hadoop-ha`: High Availability**
```
master     NameNode (active)  · ZKFC · JournalNode · ZooKeeper · ResourceManager
standby    NameNode (standby) · ZKFC · JournalNode · ZooKeeper
datanode1  DataNode · NodeManager · JournalNode · ZooKeeper
datanode2  DataNode · NodeManager
```
If the active NameNode fails, ZooKeeper detects it and the standby takes over automatically.

**`hadoop-hive`: Hive + HBase**
```
master     NameNode · ResourceManager · JobHistory · Hive · MySQL · HMaster (active)  · ZooKeeper
datanode1  DataNode · NodeManager · RegionServer · HMaster (backup)                  · ZooKeeper
datanode2  DataNode · NodeManager · RegionServer                                     · ZooKeeper
```
Kafka streaming lab (same 3 VMs): one Kafka broker per node (`broker.id` 1, 2, 3), Jupyter on master.
If the active HMaster fails, ZooKeeper detects it and the backup HMaster on datanode1 takes over.

| Node | IP |
|---|---|
| master | 192.168.56.10 |
| standby (HA only) | 192.168.56.11 |
| datanode1 | 192.168.56.12 |
| datanode2 | 192.168.56.13 |

---

## 🧰 Tools

| Tool | Role in the project |
|---|---|
| **Vagrant + VirtualBox** | Create and manage the virtual machines |
| **Ansible** | Install and configure everything (roles, templates, playbooks) |
| **HDFS** | Distributed file system: stores data in blocks across the datanodes |
| **YARN** | Resource manager: runs jobs on the cluster |
| **MapReduce** | Batch processing model: map, then shuffle, then reduce |
| **Spark (PySpark)** | In-memory processing, much faster than MapReduce for repeated work |
| **ZooKeeper** | Coordination service: elects the active NameNode (HA) and coordinates HBase |
| **Hive** | SQL queries over files in HDFS |
| **MySQL** | Stores Hive's metadata: tables, columns and locations |
| **HBase** | NoSQL column-family database on HDFS |

---

## 🚀 Quick start

**Requirements:** VirtualBox, Vagrant, Ansible, and 8–10 GB of free RAM

```bash
git checkout hadoop-hive
vagrant up                                   # about 30–45 minutes the first time
```

**Connect to master:**
```bash
vagrant ssh master
sudo -iu hadoop
```

**Check it works:**
```bash
jps                                          # lists the running services
hdfs dfsadmin -report | grep "Live datanodes"
```

---

## 🐝 This branch: `hadoop-hive`

### Start and stop
Run from the `ansible/` folder:
```bash
ansible-playbook -i inventory.ini start_cluster.yml   # HDFS → YARN → Hive → HBase
ansible-playbook -i inventory.ini stop_cluster.yml    # HBase → Hive → YARN → HDFS
```
Always stop the cluster before `vagrant halt`. For the manual commands, see [`cluster-commands.txt`](cluster-commands.txt).

### Web interfaces
| Service | URL |
|---|---|
| HDFS | http://192.168.56.10:9870 |
| YARN | http://192.168.56.10:8088 |
| JobHistory | http://192.168.56.10:19888 |
| HiveServer2 | http://192.168.56.10:10002 |
| HBase | http://192.168.56.10:16010 |

### Try it
```bash
hdfs dfs -put myfile.txt /                    # HDFS: store a file
hive -e "SHOW DATABASES;"                     # Hive: run SQL
hbase shell                                   # HBase: open the shell
```
```
create 'users', 'info'
put 'users', 'u1', 'info:name', 'Ahmad'
scan 'users'
```

### Project structure
```
.
├── Vagrantfile              # the 3 VMs (IP, RAM, CPU)
├── cluster-commands.txt     # manual start/stop commands
└── ansible/
    ├── site.yml             # full installation (run by vagrant up)
    ├── start_cluster.yml    # start all services
    ├── stop_cluster.yml     # stop all services
    ├── inventory.ini        # list of nodes
    ├── group_vars/all.yml   # versions, paths and passwords
    ├── start_kafka.yml      # Kafka lab: ZooKeeper → 3 brokers → topic INGTopic
    ├── stop_kafka.yml       # Kafka lab: brokers → ZooKeeper
    └── roles/               # common · hadoop · format_master · mysql · hive · zookeeper · hbase · kafka · jupyter
```

---

## 📝 Notes

- Passwords in `group_vars/all.yml` are for this lab only.
- `hadoop-cluster` contains the PySpark exercises (`exercises/`, `shared/`), but its playbooks don't install Spark.
- `hadoop-hive` uses Java 8; `hadoop-cluster` and `hadoop-ha` use Java 11.
- HBase HA is installed by the `zookeeper` and `hbase` roles (`HBASE_MANAGES_ZK=false`: HBase uses the shared ZooKeeper quorum).
- Kafka 3.9.2 runs in ZooKeeper mode (the last line that supports it; Kafka 4.0 uses KRaft only). Its data lives under `/kafka` in the shared ZooKeeper quorum.
- Jupyter: `jupyter notebook --ip=0.0.0.0 --port=8888 --no-browser --notebook-dir=~/notebooks`, then open `http://192.168.56.10:8888`. The notebook is `kafka_ingtopic_stream.ipynb`.
- Don't use `stop-hbase.sh`: in HBase 2.5.12 it hangs. `stop_cluster.yml` handles HBase correctly.
- Clean reinstall: `vagrant destroy -f && vagrant up`
