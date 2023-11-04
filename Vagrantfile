require "yaml"
settings = YAML.load_file "settings.yaml"

IP_SECTIONS = settings["network"]["control_ip"].match(/^([0-9.]+\.)([^.]+)$/)
# First 3 octets including the trailing dot:
IP_NW = IP_SECTIONS.captures[0]
# Last octet excluding all dots:
IP_START = Integer(IP_SECTIONS.captures[1])
NUM_WORKER_NODES = settings["nodes"]["workers"]["count"]
KUBERNETES_VERSION_FULL = settings["software"]["kubernetes"]
KUBERNETES_VERSION_SHORT = KUBERNETES_VERSION_FULL.match(/^([0-9]+\.[0-9]+)\.([0-9]+-[0-9]+)$/).captures[0]
OS = settings["software"]["os"]
DISTRO = settings["software"]["box"]
SHARED_FOLDERS = settings["shared_folders"]
CLUSTER_NAME = settings["cluster_name"]

CALICO_VERSION = settings["software"]["calico"]
POD_CIDR = settings["network"]["pod_cidr"]
SERVICE_CIDR = settings["network"]["service_cidr"]
DNS_SERVERS = settings["network"]["dns_servers"]

ENVIRONMENT = settings["environment"]

MASTER_NODE_IP = settings["network"]["control_ip"]
MASTER_NODE_CPUS = settings["nodes"]["control"]["cpu"]
MASTER_NODE_MEMORY = settings["nodes"]["control"]["memory"]

WORKER_NODE_CPUS = settings["nodes"]["workers"]["cpu"]
WORKER_NODE_MEMORY = settings["nodes"]["workers"]["memory"]

Vagrant.configure("2") do |config|
 
  # config.vm.box = "ubuntu/jammy64"

  config.vm.box = DISTRO
  config.vm.box_check_update = true

  config.vm.define "master" do |master|
    master.vm.hostname = "master-node"
    master.vm.network "private_network", ip: MASTER_NODE_IP
    if SHARED_FOLDERS
      SHARED_FOLDERS.each do |shared_folder|
        master.vm.synced_folder shared_folder["host_path"], shared_folder["vm_path"]
      end
    end
    master.vm.provider "virtualbox" do |vb|
        vb.cpus = MASTER_NODE_CPUS
        vb.memory = MASTER_NODE_MEMORY
        if CLUSTER_NAME and CLUSTER_NAME != ""
          vb.customize ["modifyvm", :id, "--groups", ("/" + CLUSTER_NAME)]
        end
    end
    master.vm.provision "ansible_local" do |ansible|
      # ansible.provisioning_path = "/home/vagrant/box/ansible"
      ansible.provisioning_path = "/vagrant/ansible/"
      ansible.playbook = "master.yml"
      ansible.extra_vars = {
        kubernetes: {
          version_short: KUBERNETES_VERSION_SHORT,
          version_full: KUBERNETES_VERSION_FULL
        },
        os: OS,
        environment: ENVIRONMENT,
        local_ip: MASTER_NODE_IP,
        calico_version: CALICO_VERSION,
        pod_cidr: POD_CIDR,
        service_cidr: SERVICE_CIDR,
        DNS_SERVERS: DNS_SERVERS.join(" ")
      }
    end
  end

  (1..NUM_WORKER_NODES).each do |i|

    config.vm.define "node0#{i}" do |node|
      node.vm.hostname = "worker-node0#{i}"
      node.vm.network "private_network", ip: IP_NW + "#{IP_START + i}"
      if SHARED_FOLDERS
        SHARED_FOLDERS.each do |shared_folder|
          node.vm.synced_folder shared_folder["host_path"], shared_folder["vm_path"]
        end
      end
      node.vm.provider "virtualbox" do |vb|
          vb.cpus = WORKER_NODE_CPUS
          vb.memory = WORKER_NODE_MEMORY
          if CLUSTER_NAME and CLUSTER_NAME != ""
            vb.customize ["modifyvm", :id, "--groups", ("/" + CLUSTER_NAME)]
          end
      end
      node.vm.provision "ansible_local" do |ansible|
        # ansible.provisioning_path = "/home/vagrant/box/ansible"
        ansible.provisioning_path = "/vagrant/ansible/"
        ansible.playbook = "worker.yml"
        ansible.extra_vars = {
           kubernetes: {
              version_short: KUBERNETES_VERSION_SHORT,
              version_full: KUBERNETES_VERSION_FULL
            },
            os: OS,
            environment: ENVIRONMENT,
            local_ip: IP_NW + "#{IP_START + i}",
            calico_version: CALICO_VERSION,
            pod_cidr: POD_CIDR,
            service_cidr: SERVICE_CIDR,
            DNS_SERVERS: DNS_SERVERS.join(" ")
        }
      end
    end

  end
end
