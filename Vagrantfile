# -*- mode: ruby -*-
# vi: set ft=ruby :


Vagrant.configure("2") do |config|

  config.vm.define "k8s1" do |k8s1|
    k8s1.vm.hostname = "k8s1"
    k8s1.vm.box = "ubuntu/focal64"
    k8s1.vm.network "private_network", ip: "192.168.33.11" 
  end

  config.vm.define "k8s2" do |k8s2|
    k8s2.vm.hostname = "k8s2"
    k8s2.vm.box = "ubuntu/focal64"
    k8s2.vm.network "private_network", ip: "192.168.33.21" 
  end

  config.vm.define "k8s3" do |k8s3|
    k8s3.vm.hostname = "k8s3"
    k8s3.vm.box = "ubuntu/focal64"
    k8s3.vm.network "private_network", ip: "192.168.33.22" 
  end    

end