# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'fileutils'
require_relative 'vagrant_rancheros_guest_plugin.rb'

Vagrant.require_version ">= 1.9.0"

CONFIG = File.join(File.dirname(__FILE__), "config.rb")

$num_instances = 1
$instance_name_prefix = "rancher"
$box_version="1.0.3"
$enable_serial_logging = false
$share_home = false
$vm_gui = false
$vm_memory = 2048
$vm_cpus = 2
$vb_cpuexecutioncap = 100
$shared_folders = {}
$forwarded_ports = {}
$expose_docker_tcp = 2375

if File.exist?(CONFIG)
  require CONFIG
end


Vagrant.configure("2") do |config|
  config.vm.provision "shell", inline: " sudo ros c set rancher.network.interfaces.eth*.dhcp true"

  config.ssh.insert_key = false
  config.ssh.forward_agent = true
  config.ssh.username = "rancher"

  config.vm.box = "kuroui/rancheros"
  config.vm.box_version = $box_version

  config.vm.provider :virtualbox do |v|
    v.check_guest_additions = false
    v.functional_vboxsf     = false
  end

  # plugin conflict
  if Vagrant.has_plugin?("vagrant-vbguest") then
    config.vbguest.auto_update = false
  end


  (1..$num_instances).each do |i|
    config.vm.define vm_name = "%s-%02d" % [$instance_name_prefix, i] do |config|
      config.vm.guest = :linux

      if $expose_docker_tcp
        config.vm.network "forwarded_port", guest: 2375, host: ($expose_docker_tcp + i - 1), host_ip: "127.0.0.1", auto_correct: true
      end

      $forwarded_ports.each do |guest, host|
        config.vm.network "forwarded_port", guest: guest, host: host, auto_correct: true
      end

      config.vm.provider :virtualbox do |vb|
        vb.gui = $vm_gui
        vb.memory = $vm_memory
        vb.cpus = $vm_cpus
        vb.customize ["modifyvm", :id, "--cpuexecutioncap", "#{$vb_cpuexecutioncap}"]
      end

      ip = "172.17.8.#{i+100}"
      config.vm.network :private_network, ip: ip

      config.vm.synced_folder ".", "/vagrant", disabled: true
    end
  end

end
