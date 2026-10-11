Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/jammy64"
  
  config.ssh.insert_key = false

  nodes = {
    "master"    => { ram: 3072, cpu: 2, ip: "192.168.56.10" },
    "datanode1" => { ram: 2048, cpu: 1, ip: "192.168.56.12" },
    "datanode2" => { ram: 2048, cpu: 1, ip: "192.168.56.13" }
  }

  nodes.each do |name, opts|
    config.vm.define name do |node|
      node.vm.hostname = name
      node.vm.network "private_network", ip: opts[:ip]

      node.vm.provider "virtualbox" do |vb|
        vb.name = name
        vb.memory = opts[:ram]
        vb.cpus = opts[:cpu]
      end

      if name == "master"
        node.vm.network "forwarded_port", guest: 9870, host: 9870   # NameNode UI
        node.vm.network "forwarded_port", guest: 8088, host: 8088   # ResourceManager UI
        node.vm.network "forwarded_port", guest: 19888, host: 19888 # JobHistory UI
        node.vm.network "forwarded_port", guest: 10002, host: 10002 # HiveServer2 Web UI
        node.vm.network "forwarded_port", guest: 10000, host: 10000
      end

      # Provision everything only once, after the LAST vm is up
      if name == "datanode2"
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