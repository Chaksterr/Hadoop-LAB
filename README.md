# Hadoop-Hive-LAB

> Automated Hadoop 3.3.6 + Hive cluster deployment using Infrastructure as Code. Spin up a fully configured 3-node Hadoop cluster with HDFS, YARN, MapReduce, and Apache Hive in minutes using Vagrant and Ansible.

## Architecture

```
Master (192.168.56.10)          DataNode1 (192.168.56.12)       DataNode2 (192.168.56.13)
├── NameNode                    ├── DataNode                    ├── DataNode
├── ResourceManager             ├── NodeManager                 └── NodeManager
├── JobHistoryServer            
├── MySQL (Hive Metastore)
└── Hive CLI
```

**Stack:** Hadoop 3.3.6 | Apache Hive 3.1.3 | MySQL 8.0 | Java 11 | Vagrant | Ansible

**Resources:** Master (3GB RAM, 2 CPUs) | DataNodes (2GB RAM, 1 CPU each)

---

## Quick Start

```bash
# Deploy cluster
vagrant up

# Verify
vagrant ssh master
jps  # Should show: NameNode, ResourceManager, JobHistoryServer
hive --version  # Verify Hive installation
```

### Web UIs
- **NameNode:** http://192.168.56.10:9870
- **ResourceManager:** http://192.168.56.10:8088
- **JobHistory:** http://192.168.56.10:19888

---

## Prerequisites

- VirtualBox
- Vagrant
- Ansible
- 7GB+ free RAM

---

## Repository Structure

```
.
├── Vagrantfile                 # VM definitions
├── ansible/
│   ├── site.yml                # Main playbook (setup + start)
│   ├── start_cluster.yml       # Start services
│   ├── stop_cluster.yml        # Stop services
│   ├── inventory.ini           # Node definitions
│   ├── group_vars/
│   │   └── all.yml             # Configuration variables
│   ├── common/                 # Java, users, SSH
│   ├── hadoop/                 # Hadoop installation
│   ├── hive/                   # Hive installation
│   ├── mysql/                  # MySQL metastore setup
│   └── format_master/          # NameNode formatting
```

---

## Managing the Cluster

### Vagrant Commands
```bash
vagrant up              # Start VMs
vagrant halt            # Stop VMs
vagrant destroy -f      # Delete VMs
vagrant ssh master      # SSH to master
vagrant status          # Check status
```

### Ansible Playbooks
```bash
cd ansible

# Full setup
ansible-playbook -i inventory.ini site.yml

# Start/Stop services
ansible-playbook -i inventory.ini start_cluster.yml
ansible-playbook -i inventory.ini stop_cluster.yml
```

### Manual Service Control
```bash
# Check services
jps

# Start Hadoop services (on master)
sudo -u hadoop /home/hadoop/hadoop/bin/hdfs --daemon start namenode
sudo -u hadoop /home/hadoop/hadoop/bin/yarn --daemon start resourcemanager
sudo -u hadoop /home/hadoop/hadoop/bin/mapred --daemon start historyserver

# Stop services (replace 'start' with 'stop')
```

---

## HDFS Commands

```bash
# List files
hdfs dfs -ls /

# Create directory
hdfs dfs -mkdir -p /user/hadoop/input

# Upload file
hdfs dfs -put file.txt /user/hadoop/input/

# Download file
hdfs dfs -get /user/hadoop/output/result.txt ./

# View file
hdfs dfs -cat /user/hadoop/output/part-00000

# Delete
hdfs dfs -rm -r /user/hadoop/output

# Cluster status
hdfs dfsadmin -report
```

---

## Hive Usage

### Starting Hive
```bash
# SSH to master
vagrant ssh master

# Start Hive CLI
hive

# Or use beeline (recommended)
beeline -u jdbc:hive2://
```

### Basic Hive Commands
```sql
-- Show databases
SHOW DATABASES;

-- Create database
CREATE DATABASE mydb;
USE mydb;

-- Create table
CREATE TABLE users (
    id INT,
    name STRING,
    age INT
) ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE;

-- Load data from HDFS
LOAD DATA INPATH '/user/hadoop/input/users.csv' INTO TABLE users;

-- Query data
SELECT * FROM users WHERE age > 25;

-- Show tables
SHOW TABLES;
```

---

## User Management

### Default Users
- **vagrant** - Auto-switches to hadoop on SSH
- **hadoop** - Main service user with sudo

---

## Configuration

Edit `ansible/group_vars/all.yml`:
```yaml
hadoop_version: "3.3.6"
hive_version: "3.1.3"
java_version: "11"
hadoop_home: "/home/hadoop/hadoop"
hive_home: "/home/hadoop/hive"
namenode_host: "master"
namenode_port: "9000"
```

### Hadoop Configuration Templates
Located in `ansible/hadoop/templates/`:
- `core-site.xml.j2`
- `hdfs-site.xml.j2`
- `yarn-site.xml.j2`
- `mapred-site.xml.j2`
- `workers.j2`

### Hive Configuration Templates
Located in `ansible/hive/templates/`:
- `hive-site.xml.j2` - Metastore and database configuration

---

## Troubleshooting

### Services Not Starting
```bash
# Check Hadoop logs
tail -f /home/hadoop/hadoop/logs/*.log

# Check Hive logs
tail -f /home/hadoop/hive/logs/*.log

# Verify Java
java -version

# Restart services
cd ansible
ansible-playbook -i inventory.ini stop_cluster.yml
ansible-playbook -i inventory.ini start_cluster.yml
```

### NameNode Safe Mode
```bash
hdfs dfsadmin -safemode get
hdfs dfsadmin -safemode leave  # If stuck
```

### DataNodes Not Connecting
```bash
# Check logs
tail -f /home/hadoop/hadoop/logs/hadoop-hadoop-datanode-*.log

# Test connectivity
ping master
ping datanode1
```

### Hive Metastore Connection Issues
```bash
# Check MySQL service
sudo systemctl status mysql

# Test MySQL connection
mysql -u hive -p hive_metastore

# Verify Hive metastore schema
mysql -u hive -p -e "USE hive_metastore; SHOW TABLES;"
```

### Re-format NameNode (⚠️ Erases all data!)
```bash
cd ansible
ansible-playbook -i inventory.ini stop_cluster.yml

vagrant ssh master
sudo -u hadoop /home/hadoop/hadoop/bin/hdfs namenode -format -force

cd ansible
ansible-playbook -i inventory.ini start_cluster.yml
```

---

## What's Different from Basic Hadoop?

This setup extends a standard Hadoop cluster with:
- **Apache Hive** - SQL-like interface for Hadoop
- **MySQL Metastore** - Persistent metadata storage for Hive
- **Automated Hive Setup** - Pre-configured with metastore connection
- **Beeline Support** - Modern Hive CLI interface

Perfect for learning Hive and running SQL queries on Hadoop without manual configuration!
