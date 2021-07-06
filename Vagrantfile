# -*- mode: ruby -*-
# vi: set ft=ruby :


Vagrant.configure("2") do |config|

  config.vm.provider :virtualbox do |v|
    v.linked_clone = true
  end

  (1..1).each do |i|
    config.vm.define ("m%02d" % i) do |node|
      node.vm.hostname = "m%02d" % i
      node.vm.box = "ubuntu/focal64"
      node.vm.network "private_network", ip: "192.168.33.#{10+i}"
    end
  end

  (1..2).each do |i|
    config.vm.define ("w%02d" % i) do |node|
      node.vm.hostname = "w%02d" % i
      node.vm.box = "ubuntu/focal64"
      node.vm.network "private_network", ip: "192.168.33.#{20+i}"
    end
  end

end
