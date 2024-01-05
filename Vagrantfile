# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|
  config.vm.box = "almalinux/9"

  config.vm.provider "virtualbox" do |v|
    v.memory = 2048
    v.cpus = 1
  end

  config.vm.define "otusrpm" do |otusrpm|
    otusrpm.vm.network "private_network", ip: "192.168.56.60", virtualbox__intnet: "net1"
    otusrpm.vm.hostname = "otusrpm"
  end

end
