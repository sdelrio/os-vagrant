# -*- mode: ruby -*-
# vi: set ft=ruby :

require_relative 'vagrant_rancheros_guest_plugin.rb'

# To enable rsync folder share change to false
$rsync_folder_disabled = true
$number_of_nodes = 1
$vm_mem = "1024"
$vb_gui = false
$node_script = <<SCRIPT
uname -a
SCRIPT
$rs_vm_mem = "1024"
$rs_vb_gui = false
$rs_script = <<SCRIPT
mkdir -p /opt/rancher/mysql
docker run -d --restart=always -p 80:8080 -v /opt/rancher/mysql:/var/lib/mysql rancher/server
SCRIPT

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure(2) do |config|
  config.vm.box   = "rancherio/rancheros"
  config.vm.box_version = ">=0.4.1"

  rhostname = "rancher-server"
  config.vm.define rhostname do |node|
    node.vm.provider "virtualbox" do |vb|
      vb.memory = $rs_vm_mem
      vb.gui =$rs_vm_gui
    end

    ip = "172.19.8.10"
    node.vm.network "private_network", ip: ip
    node.vm.hostname = rhostname

    node.vm.synced_folder ".", "/opt/rancher", type: "rsync",
      rsync__exclude: ".git/", rsync__args: ["--verbose", "--archive", "--delete", "--copy-links"],
      disabled: $rsync_folder_disabled
    node.vm.provision "shell", inline: $rs_script
  end

  (1..$number_of_nodes).each do |i|
    hostname = "rancher-%02d" % i

    config.vm.define hostname do |node|
        node.vm.provider "virtualbox" do |vb|
            vb.memory = $vm_mem
            vb.gui = $vb_gui
        end

        ip = "172.19.8.#{i+100}"
        node.vm.network "private_network", ip: ip
        node.vm.hostname = hostname

        # Disabling compression because OS X has an ancient version of rsync installed.
        # Add -z or remove rsync__args below if you have a newer version of rsync on your machine.
        node.vm.synced_folder ".", "/opt/rancher", type: "rsync",
            rsync__exclude: ".git/", rsync__args: ["--verbose", "--archive", "--delete", "--copy-links"],
            disabled: $rsync_folder_disabled

        node.vm.provision "shell", inline: $node_script
    end
  end
end
