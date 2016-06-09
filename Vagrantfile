# -*- mode: ruby -*-
# vi: set ft=ruby :

require_relative 'vagrant_rancheros_guest_plugin.rb'

# IP addresses for rancher server and base net for nodes
$base_net = "172.19.8"
$first_ip = 100
$rs_ip = $base_net + ".#{$first_ip}"

# To enable rsync folder share change to false
$rsync_folder_disabled = true
$number_of_nodes = 1
$vm_mem = "1024"
$vb_gui = false

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

    node.vm.network "private_network", ip: $rs_ip
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

        ip = "#{$base_net}.#{i+$first_ip}"
        node.vm.network "private_network", ip: ip
        node.vm.hostname = hostname

        # Disabling compression because OS X has an ancient version of rsync installed.
        # Add -z or remove rsync__args below if you have a newer version of rsync on your machine.
        node.vm.synced_folder ".", "/opt/rancher", type: "rsync",
            rsync__exclude: ".git/", rsync__args: ["--verbose", "--archive", "--delete", "--copy-links"],
            disabled: $rsync_folder_disabled

        node.vm.provision "shell", inline: <<-SCRIPT
          echo "show hostname"
          sudo ros config get hostname  2>&1
        SCRIPT
    end
  end
end
