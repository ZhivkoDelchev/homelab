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

ENVIRONMENT = settings["environment"]

Vagrant.configure("2") do |config|
 
  # config.vm.box = "ubuntu/jammy64"

  config.vm.box = settings["software"]["box"]
  config.vm.box_check_update = true

  config.vm.define "master" do |master|
    master.vm.hostname = "master-node"
    master.vm.network "private_network", ip: settings["network"]["control_ip"]
    if settings["shared_folders"]
      settings["shared_folders"].each do |shared_folder|
        master.vm.synced_folder shared_folder["host_path"], shared_folder["vm_path"]
      end
    end
    master.vm.provider "virtualbox" do |vb|
        vb.cpus = settings["nodes"]["control"]["cpu"]
        vb.memory = settings["nodes"]["control"]["memory"]
        if settings["cluster_name"] and settings["cluster_name"] != ""
          vb.customize ["modifyvm", :id, "--groups", ("/" + settings["cluster_name"])]
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
        local_ip: settings["network"]["control_ip"]
      }
    end
    # master.vm.provision "shell",
    #   env: {
    #     "DNS_SERVERS" => settings["network"]["dns_servers"].join(" "),
    #     "ENVIRONMENT" => settings["environment"],
    #     "KUBERNETES_VERSION" => settings["software"]["kubernetes"],
    #     "OS" => settings["software"]["os"]
    #   },
    #   path: "scripts/common.sh"
    # master.vm.provision "shell",
    #   env: {
    #     "CALICO_VERSION" => settings["software"]["calico"],
    #     "CONTROL_IP" => settings["network"]["control_ip"],
    #     "POD_CIDR" => settings["network"]["pod_cidr"],
    #     "SERVICE_CIDR" => settings["network"]["service_cidr"]
    #   },
    #   path: "scripts/master.sh"
  end

  # (1..NUM_WORKER_NODES).each do |i|

  #   config.vm.define "node0#{i}" do |node|
  #     node.vm.hostname = "worker-node0#{i}"
  #     node.vm.network "private_network", ip: IP_NW + "#{IP_START + i}"
  #     if settings["shared_folders"]
  #       settings["shared_folders"].each do |shared_folder|
  #         node.vm.synced_folder shared_folder["host_path"], shared_folder["vm_path"]
  #       end
  #     end
  #     node.vm.provider "virtualbox" do |vb|
  #         vb.cpus = settings["nodes"]["workers"]["cpu"]
  #         vb.memory = settings["nodes"]["workers"]["memory"]
  #         if settings["cluster_name"] and settings["cluster_name"] != ""
  #           vb.customize ["modifyvm", :id, "--groups", ("/" + settings["cluster_name"])]
  #         end
  #     end
  #     node.vm.provision "ansible_local" do |ansible|
  #       # ansible.provisioning_path = "/home/vagrant/box/ansible"
  #       ansible.provisioning_path = "/vagrant/ansible/"
  #       ansible.playbook = "worker.yml"
  #       ansible.extra_vars = {
  #          kubernetes: {
  #             version_short: KUBERNETES_VERSION_SHORT,
  #             version_full: KUBERNETES_VERSION_FULL
  #           },
  #           os: OS,
  #           environment: ENVIRONMENT,
  #           local_ip: IP_NW + "#{IP_START + i}"
  #       }
  #     end
  #     # node.vm.provision "shell",
  #     #   env: {
  #     #     "DNS_SERVERS" => settings["network"]["dns_servers"].join(" "),
  #     #     "ENVIRONMENT" => settings["environment"],
  #     #     "KUBERNETES_VERSION" => settings["software"]["kubernetes"],
  #     #     "OS" => settings["software"]["os"]
  #     #   },
  #     #   path: "scripts/common.sh"
  #     # node.vm.provision "shell", path: "scripts/node.sh"

  #     # # Only install the dashboard after provisioning the last worker (and when enabled).
  #     # if i == NUM_WORKER_NODES and settings["software"]["dashboard"] and settings["software"]["dashboard"] != ""
  #     #   node.vm.provision "shell", path: "scripts/dashboard.sh"
  #     # end
  #   end

  # end
end
