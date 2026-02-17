Vagrant.configure("2") do |config|

  config.vm.box = "ubuntu/jammy64"
  
  config.vm.synced_folder "./shared", "/vagrant_shared", create: true

  
  nodes = {
    "master" => { ram: 3072, cpu: 2, ip: "192.168.56.10" },
    "standby" => {ram:2048,cpu:1,ip:"192.168.56.11"},
    "datanode1" => { ram: 2048, cpu: 1, ip: "192.168.56.12" },
    "datanode2" => { ram: 2048, cpu: 1, ip: "192.168.56.13" }
  }

  nodes.each do |name, opts|

    config.vm.define name do |node|
      node.vm.hostname = name
      
      # Configure private network for cluster communication
      node.vm.network "private_network", ip: opts[:ip]
      
    if name == "master"
      node.vm.network "forwarded_port", guest: 9870, host: 9870   # NameNode Web UI
      node.vm.network "forwarded_port", guest: 8088, host: 8088   # ResourceManager Web UI
      node.vm.network "forwarded_port", guest: 19888, host: 19888 # JobHistory Server UI
      node.vm.network "forwarded_port", guest: 8480, host: 8480   # JournalNode Web UI (ADD THIS)
      node.vm.network "forwarded_port", guest: 8080, host: 8080   # ZooKeeper Admin Server (ADD THIS)
    end
    
      if name =="standby"
        node.vm.network "forwarded_port", guest: 9870, host: 9871 # Standby NameNode Web UI (different host port)
        node.vm.network "forwarded_port", guest:8480, host:8481 # JournalNode Web UI
        node.vm.network "forwarded_port", guest: 8080, host: 8081   # ZooKeeper Admin Server
      end

      if name == "datanode1"
        node.vm.network "forwarded_port", guest: 9864, host: 9864   # DataNode1 Web UI
        node.vm.network "forwarded_port", guest: 8480, host: 8482   # JournalNode Web UI (ADD THIS)
        node.vm.network "forwarded_port", guest: 8080, host: 8082   # ZooKeeper Admin Server (ADD THIS)
      end
      
      if name == "datanode2"
        node.vm.network "forwarded_port", guest: 9864, host: 9865   # DataNode2 Web UI (different host port)
      end

      node.vm.provider "virtualbox" do |vb|
        vb.name = name
        vb.memory = opts[:ram]
        vb.cpus = opts[:cpu]
      end
      
      # Only provision after all VMs are created

      if name == "standby"  # Changed from datanode2
        node.vm.provision "ansible" do |ansible|
        ansible.playbook = "ansible/site.yml"
        ansible.inventory_path = "ansible/inventory.ini"
        ansible.limit = "all"
        ansible.verbose = "v"
      end
      end
    end
  end

end
