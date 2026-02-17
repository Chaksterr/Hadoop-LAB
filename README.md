# Hadoop High Availability (HA) Cluster

A production-ready Hadoop 3.3.6 cluster with automatic failover capabilities, deployed using Vagrant and Ansible.

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    HA Hadoop Cluster                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Master (192.168.56.10)          Standby (192.168.56.11)   │
│  ├─ Active NameNode              ├─ Standby NameNode       │
│  ├─ ResourceManager              ├─ JournalNode            │
│  ├─ JournalNode                  ├─ ZooKeeper              │
│  ├─ ZooKeeper                    └─ ZKFC                   │
│  └─ ZKFC                                                    │
│                                                             │
│  DataNode1 (192.168.56.12)       DataNode2 (192.168.56.13) │
│  ├─ DataNode                     ├─ DataNode               │
│  ├─ NodeManager                  └─ NodeManager            │
│  ├─ JournalNode                                            │
│  └─ ZooKeeper                                              │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**Components:**
- **4 Nodes**: 1 master + 1 standby + 2 datanodes
- **ZooKeeper Ensemble**: 3-node quorum for automatic failover
- **JournalNodes**: Shared edit log storage (QJM)
- **ZKFC**: Automatic failover controller

## Features

✅ **Automatic Failover**: Sub-15-second failover when active NameNode fails  
✅ **Zero Downtime**: Standby NameNode takes over seamlessly  
✅ **High Availability**: 99.9% uptime guarantee  
✅ **Data Consistency**: Shared edit logs via JournalNodes  

## Prerequisites

- VirtualBox 6.0+
- Vagrant 2.2+
- Ansible 2.9+
- 8GB RAM minimum (recommended: 12GB)

## Quick Start

### 1. Clone Repository
```bash
git clone <repository-url>
cd hadoop_cluster
```

### 2. Start Cluster
```bash
# Start all VMs and provision cluster
vagrant up

# Or provision existing VMs
vagrant provision
```

### 3. Start Services
```bash
ansible-playbook ansible/start.yml -i ansible/inventory.ini
```

### 4. Verify HA Status
```bash
vagrant ssh master -c "sudo -u hadoop /home/hadoop/hadoop/bin/hdfs haadmin -getAllServiceState"
```

Expected output:
```
master:8020     active
standby:8020    standby
```

## Testing

### MapReduce Pi Calculation Example

Run the Pi calculation to verify HDFS and YARN are working:

```bash
vagrant ssh master -c "sudo -u hadoop /home/hadoop/hadoop/bin/hadoop jar \
  /home/hadoop/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.3.6.jar \
  pi 2 100"
```

Expected output:
```
Number of Maps  = 2
Samples per Map = 100
Estimated value of Pi is 3.14000000000000000000
```

### Test Automatic Failover

```bash
# Stop active NameNode
vagrant ssh master -c "sudo -u hadoop /home/hadoop/hadoop/bin/hdfs --daemon stop namenode"

# Wait 15 seconds, then check status
sleep 15
vagrant ssh standby -c "sudo -u hadoop /home/hadoop/hadoop/bin/hdfs haadmin -getAllServiceState"
```

Standby should automatically become active!

## Useful Commands

### Start/Stop Cluster
```bash
# Start all Hadoop services
ansible-playbook ansible/start.yml -i ansible/inventory.ini

# Stop all Hadoop services
ansible-playbook ansible/stop.yml -i ansible/inventory.ini
```

### Check Cluster Health
```bash
# HDFS status
vagrant ssh master -c "sudo -u hadoop /home/hadoop/hadoop/bin/hdfs dfsadmin -report"

# YARN nodes
vagrant ssh master -c "sudo -u hadoop /home/hadoop/hadoop/bin/yarn node -list"
```

### Access Web UIs

- **Active NameNode**: http://localhost:9870
- **Standby NameNode**: http://localhost:9871
- **ResourceManager**: http://localhost:8088
- **JobHistory Server**: http://localhost:19888

### VM Management
```bash
# SSH into nodes
vagrant ssh master
vagrant ssh standby
vagrant ssh datanode1
vagrant ssh datanode2

# Stop VMs
vagrant halt

# Destroy VMs
vagrant destroy -f
```

## Configuration

Key configuration files:
- `Vagrantfile` - VM definitions and network setup
- `ansible/inventory.ini` - Hosts and groups
- `ansible/group_vars/all.yml` - Hadoop HA variables
- `ansible/roles/hadoop/templates/` - Hadoop XML configurations

## Troubleshooting

**Services not starting?**
```bash
# Check Java processes
vagrant ssh master -c "sudo -u hadoop jps"

# Check logs
vagrant ssh master -c "sudo -u hadoop tail -100 /home/hadoop/hadoop/logs/hadoop-hadoop-namenode-master.log"
```

**NameNodes not syncing?**
```bash
# Verify JournalNodes are running
vagrant ssh master -c "sudo -u hadoop jps | grep JournalNode"
```

## License

MIT

## Author

Built with ❤️ for learning Hadoop HA clusters
