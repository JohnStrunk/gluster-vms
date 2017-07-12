# -*- mode: ruby -*-
# vi: set ft=ruby :

CLIENTS = 2
STORAGENODES = 6
DISKS_PER_NODE = 1
DISKSIZE = '30G'

# Note: libvirt provider options:
#   https://github.com/vagrant-libvirt/vagrant-libvirt#provider-options

Vagrant.configure("2") do |config|
  #config.vm.box = "fedora/25-atomic-host"
  config.vm.box = "fedora/25-cloud-base"

  config.vm.provider "libvirt" do |provider|
    provider.cpus = "2"
    provider.memory = "1024"
  end

  config.vm.synced_folder ".", "/vagrant", disabled: true
  config.vm.synced_folder "./scripts", "/scripts", type: "sshfs"

  #-- install python so we can use ansible
  config.vm.provision "shell",
    inline: "dnf install -y python"

  (0..CLIENTS-1).each do |i|
    config.vm.define "client#{i}" do |node|
      node.vm.hostname = "client#{i}"
    end
  end

  (0..STORAGENODES-1).each do |i|
    config.vm.define "storage#{i}" do |node|
      node.vm.hostname = "storage#{i}"
      node.vm.provider "libvirt" do |provider|
        for d in 0..DISKS_PER_NODE-1 do
          provider.storage :file, :size => DISKSIZE
        end
      end
      # Only set up provisioning on the last worker node so ansible only gets
      # called once during the provisioning step.
      if i == STORAGENODES-1
        node.vm.provision :ansible do |ansible|
          #ansible.extra_vars = {
          #  "master_ip" => MASTER_IP
          #}
          ansible.groups = {
            "clients" => (0..CLIENTS-1).map {|j| "client#{j}"},
            "storagenodes" => (0..STORAGENODES-1).map {|j| "storage#{j}"}
          }
          ansible.limit = "all"
          ansible.playbook = "ansible/initial_provisioning.yml"
          ansible.verbose = true
        end
      end
    end
  end
end
