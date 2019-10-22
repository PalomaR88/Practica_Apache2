# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  config.vm.define :serapache do |serapache|
    serapache.vm.box = "debian/buster64"
    serapache.vm.hostname = "servidor"
    serapache.vm.network :public_network,:bridge=>"enp2s0"
    #nodo1.vm.network :private_network, ip: "192.168.100.1", virtualbox__intnet:"mired1"
    #nodo1.vm.network :private_network, ip: "192.168.200.1", virtualbox__intnet:"mired2"
  end

#  config.vm.define :nodo2 do |nodo2|
 #   nodo2.vm.box = "debian/buster64"
  #  nodo2.vm.hostname = "nodolan1"
   # nodo2.vm.network :private_network, type:"dhcp", virtualbox__intnet:"mired1"
  #end
  #config.vm.define :nodo3 do |nodo3|
   # nodo3.vm.box = "debian/buster64"
    #nodo3.vm.hostname = "nodolan2"
    #nodo3.vm.network :private_network, type:"dhcp", virtualbox__intnet:"mired2"
  #end
end

