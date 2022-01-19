IMAGE_NAME="bento/ubuntu-18.04"
MEMORY_SIZE_IN_GB=2
CPU_COUNT=2
MASTER_NODE_COUNT=1
WORKER_NODE_COUNT=2
MASTER_NODE_IP_START="10.0.0.10"
WORKER_NODE_IP_START="10.0.0.20"

Vagrant.configure("2") do |config|

  # set variables
  master_node_ip = ''
  worker_node_ip = ''

  config.vm.box = IMAGE_NAME

  config.vm.provider "virtualbox" do |vb|

    vb.memory = 1024 * MEMORY_SIZE_IN_GB
    vb.cpus = CPU_COUNT

  end

  config.vm.provision "shell", path: "pre"

  config.vm.provision "shell", path: "install-docker"
  config.vm.provision "shell", path: "install-kube-tools"

  config.vm.provision "shell", path: "post"

  (1..MASTER_NODE_COUNT).each do |i|
    config.vm.define "m" do |master|

      master_node_ip = "#{MASTER_NODE_IP_START}#{i}"
      master.vm.network "private_network", ip: "#{master_node_ip}"
      master.vm.hostname = "m"

      # init master node.
      master.vm.provision "shell", path: "init-master-node", env: {"NODE_IP" => "#{master_node_ip}"}

      # prepare kubectl for vagrant user
      master.vm.provision "shell", privileged: false, path: "prepare-kubectl"

      # prepare kubectl for root user
      master.vm.provision "shell", privileged: true, path: "prepare-kubectl"

      # install cni.
      master.vm.provision "shell", path: "install-cni"

    end
  end

  (1..WORKER_NODE_COUNT).each do |i|
    config.vm.define "n#{i}" do |node|

      worker_node_ip = "#{WORKER_NODE_IP_START}#{i}"
      node.vm.network "private_network", ip: "#{worker_node_ip}"
      node.vm.hostname = "n#{i}"

      # init slave node.
      node.vm.provision "shell", path: "init-slave-node", env: {"NODE_IP" => "#{worker_node_ip}"}

    end
  end
end
