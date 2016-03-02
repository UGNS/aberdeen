# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'yaml'

current_dir = File.dirname(File.expand_path(__FILE__))
configs_file = "#{current_dir}/config.yaml"
begin
  pconfigs = File.exists?(configs_file) ? YAML.load_file(configs_file) : Hash.new {}
  minions = pconfigs.has_key?('minions') ? pconfigs['minions'] : Hash.new {}
end

ENV["LC_ALL"] = "en_US.UTF-8"

# Static IP address for the SaltStack master
SALT_MASTER = "192.168.33.10"

Vagrant.configure(2) do |config|

  config.vm.define "master", primary: true do |master|
    master.vm.box = "debian/contrib-jessie64"
    master.vm.hostname = "#{ENV['USER']}-master.local"
    master.vm.network "private_network", ip: SALT_MASTER

    master.vm.network "forwarded_port", guest: 4505, host: 4505, protocol: "tcp"
    master.vm.network "forwarded_port", guest: 4506, host: 4506, protocol: "tcp"
    master.vm.network "forwarded_port", guest: 8000, host: 18000, protocol: "tcp"

    # Sync salt pillars directory if defined
    if pconfigs.has_key?('pillars')
      master.vm.synced_folder pconfigs['pillars'], "/srv/pillar"
    end

    # Sync salt states directory if defined
    if pconfigs.has_key?('states')
      master.vm.synced_folder pconfigs['states'], "/srv/salt"
    end

    # Sync salt reactors directory if defined
    if pconfigs.has_key?('reactors')
      master.vm.synced_folder pconfigs['reactors'], "/srv/reactor"
    end

    # Sync additional file_roots directories if defined
    if pconfigs.has_key?('file_roots')
      pconfigs['file_roots'].each do |root, path|
        master.vm.synced_folder "#{path}", "/srv/file_roots/#{root}"
      end
    end

    master.vm.provider "virtualbox" do |vb|
      vb.memory = 2048
      vb.cpus = 2
    end

    master.vm.provision "shell", inline: <<-SHELL
      sudo apt-get update
      sudo apt-get install -y git
      sudo mkdir -p /srv/formulas
      cd /srv/formulas
      sudo git clone https://github.com/saltstack-formulas/salt-formula.git
      sudo git clone https://github.com/saltstack-formulas/sudoers-formula.git
      sudo git clone https://github.com/saltstack-formulas/users-formula.git
      sudo git clone https://github.com/saltstack-formulas/ntp-formula.git
      sudo git clone https://github.com/saltstack-formulas/cert-formula.git
    SHELL

    master.vm.provision :salt do |salt|
      salt.install_master = true
      salt.install_type = 'stable'
      salt.run_highstate = true

      salt.minion_config = "salt/minion"
      salt.minion_key = "salt/keys/minion.pem"
      salt.minion_pub = "salt/keys/minion.pub"

      salt.grains_config = "salt/grains/master"
      salt.master_config = "salt/master"
      salt.master_key = "salt/keys/master.pem"
      salt.master_pub = "salt/keys/master.pub"

      salt.seed_master = {
        "#{ENV['USER']}-master.local": salt.minion_pub,
        "#{ENV['USER']}-debian.local": salt.minion_pub,
        "#{ENV['USER']}-ubuntu.local": salt.minion_pub,
      }
    end
  end

  # Deploy optional Debian 8 minion
  config.vm.define "debian", autostart: minions.has_key?('debian') ? minions['debian'] : false do |minion|
    minion.vm.box = "debian/jessie64"
    minion.vm.hostname = "#{ENV['USER']}-debian.local"
    minion.vm.network "private_network", type: "dhcp"

    minion.vm.provision :salt do |salt|
      salt.install_master = false
      salt.install_type = "stable"
      salt.run_highstate = true

      grains = "salt/grains/#{ENV['USER']}-ubuntu"
      if File.exists?(grains)
        salt.grains_config = grains
      end
      salt.minion_config = "salt/minion"
      salt.minion_key = "salt/keys/minion.pem"
      salt.minion_pub = "salt/keys/minion.pub"
    end
  end

  # Deploy optional Ubuntu 14.04 LTS minion
  config.vm.define "ubuntu", autostart: minions.has_key?('ubuntu') ? minions['ubuntu'] : false do |minion|
    minion.vm.box = "ubuntu/trusty64"
    minion.vm.hostname = "#{ENV['USER']}-ubuntu.local"
    minion.vm.network "private_network", type: "dhcp"

    minion.vm.provision :salt do |salt|
      salt.install_master = false
      salt.install_type = "stable"
      salt.run_highstate = true

      grains = "salt/grains/#{ENV['USER']}-ubuntu"
      if File.exists?(grains)
        salt.grains_config = grains
      end
      salt.minion_config = "salt/minion"
      salt.minion_key = "salt/keys/minion.pem"
      salt.minion_pub = "salt/keys/minion.pub"
    end
  end
end
