# -*- mode: ruby -*-
# vi: set ft=ruby :

vm_image = "nobreak-labs/rocky-9"
vm_subnet1 = "192.168.56."  # 외부 접속용
vm_subnet2 = "192.168.57."  # 웹/NFS 서비스
vm_subnet3 = "192.168.58."  # 데이터베이스/스토리지

Vagrant.configure("2") do |config|
  config.vm.synced_folder ".", "/vagrant", disabled: true
  
  # 로드밸런서
  config.vm.define "lb01" do |node|
    node.vm.box = vm_image
    node.vm.provider "virtualbox" do |vb|
      vb.name = "lb01"
      vb.cpus = 1
      vb.memory = 1024
    end
    node.vm.network "private_network", ip: vm_subnet1 + "10", nic_type: "virtio"
    node.vm.network "private_network", ip: vm_subnet2 + "10", nic_type: "virtio"
    node.vm.hostname = "lb01"
  end
  
  # 웹서버 1
  config.vm.define "web01" do |node|
    node.vm.box = vm_image
    node.vm.provider "virtualbox" do |vb|
      vb.name = "web01"
      vb.cpus = 2
      vb.memory = 1024
    end
    node.vm.network "private_network", ip: vm_subnet2 + "11", nic_type: "virtio"
    node.vm.network "private_network", ip: vm_subnet3 + "11", nic_type: "virtio"
    node.vm.hostname = "web01"
  end
  
  # 웹서버 2
  config.vm.define "web02" do |node|
    node.vm.box = vm_image
    node.vm.provider "virtualbox" do |vb|
      vb.name = "web02"
      vb.cpus = 2
      vb.memory = 1024
    end
    node.vm.network "private_network", ip: vm_subnet2 + "12", nic_type: "virtio"
    node.vm.network "private_network", ip: vm_subnet3 + "12", nic_type: "virtio"
    node.vm.hostname = "web02"
  end
  
  # NFS 서버
  config.vm.define "nfs01" do |node|
    node.vm.box = vm_image
    node.vm.provider "virtualbox" do |vb|
      vb.name = "nfs01"
      vb.cpus = 1
      vb.memory = 1024
    end
    node.vm.network "private_network", ip: vm_subnet2 + "20", nic_type: "virtio"
    node.vm.hostname = "nfs01"
  end
  
  # Primary DB 서버
  config.vm.define "db01" do |node|
    node.vm.box = vm_image
    node.vm.provider "virtualbox" do |vb|
      vb.name = "db01"
      vb.cpus = 2
      vb.memory = 1024
    end
    node.vm.network "private_network", ip: vm_subnet3 + "13", nic_type: "virtio"
    node.vm.hostname = "db01"
  end
  
  # Secondary DB 서버
  config.vm.define "db02" do |node|
    node.vm.box = vm_image
    node.vm.provider "virtualbox" do |vb|
      vb.name = "db02"
      vb.cpus = 2
      vb.memory = 1024
    end
    node.vm.network "private_network", ip: vm_subnet3 + "14", nic_type: "virtio"
    node.vm.hostname = "db02"
  end
  
  # iSCSI 스토리지 서버 (storage01)
  config.vm.define "storage01" do |node|
    node.vm.box = vm_image
    node.vm.provider "virtualbox" do |vb|
      vb.name = "storage01"
      vb.cpus = 1
      vb.memory = 1024
    end
    node.vm.network "private_network", ip: vm_subnet3 + "20", nic_type: "virtio"
    node.vm.hostname = "storage01"
  end
end