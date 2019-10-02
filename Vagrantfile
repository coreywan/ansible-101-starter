Vagrant.configure("2") do |config|
  config.vm.provision "shell", inline: "sudo yum install -y python python-pip libselinux-python"

  config.vm.box = "fedora/28-cloud-base"
  config.vm.provider "virtualbox" do |v|
    v.memory = 1024
    v.cpus = 2
    v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    v.customize ["modifyvm", :id, "--natdnsproxy1", "on"]
  end
  config.vm.define "web01" do |config|
    config.vm.hostname = "web01"
    config.vm.network :private_network, ip: "192.168.10.11"
  end

  config.vm.define "web02" do |config|
    config.vm.hostname = "web02"
    config.vm.network :private_network, ip: "192.168.10.12"
  end

  config.vm.define "db01" do |config|
    config.vm.hostname = "db01"
    config.vm.network :private_network, ip: "192.168.10.13"
  end

  config.vm.define "db02" do |config|
    config.vm.hostname = "db02"
    config.vm.network :private_network, ip: "192.168.10.14"
  end
end