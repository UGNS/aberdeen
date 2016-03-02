# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|
  config.vm.box = "debian/contrib-jessie64"

  # config.vm.network "forwarded_port", guest: 80, host: 8080

  config.vm.network "private_network", ip: "192.168.33.10"

  # config.vm.synced_folder "../data", "/vagrant_data"

  config.vm.provider "virtualbox" do |vb|
    vb.memory = 1024
    vb.cpus = 2
  end

  config.vm.provision "shell", inline: <<-SHELL
    sudo apt-get update
    sudo apt-get install -y git
    sudo mkdir -p /srv/formulas
    sudo git clone https://github.com/saltstack-formulas/salt-formula.git /srv/formulas/salt-formula
  SHELL

  # config.vm.provision :salt do |salt|
  #   salt.install_master = true
  #   salt.install_type = 'stable'
  # end
end
