# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"
BOX_NAME = "centos/7"
BOX_FILE_URL = ""

ANSIBLE_VERBOSE = ENV['ANSIBLE_VERBOSE'] == nil ? "v" : ENV['ANSIBLE_VERBOSE']

# -------------------------- Settings for all hosts --------------------------
# salt master
SALT_MASTER1_HOSTNAME = "salt-master1"
SALT_MASTER1_IP = ENV['SALT_MASTER1_IP'] == nil ? "192.168.11.10" : ENV['SALT_MASTER1_IP']
SALT_MASTER1_CPU = "2"
SALT_MASTER1_MEM = "2048"

# salt minion1
SALT_MINION1_HOSTNAME = "salt-minion1"
SALT_MINION1_IP = ENV['SALT_MINION1_IP'] == nil ? "192.168.22.10" : ENV['SALT_MINION1_IP']
SALT_MINION1_CPU = "4"
SALT_MINION1_MEM = "4096"

# salt minion2
SALT_MINION2_HOSTNAME = "salt-minion2"
SALT_MINION2_IP = ENV['SALT_MINION2_IP'] == nil ? "192.168.22.11" : ENV['SALT_MINION2_IP']
SALT_MINION2_CPU = "4"
SALT_MINION2_MEM = "4096"

# -------------------- VM settings for 'vargrant up' stage --------------------
VMS = {
  SALT_MASTER1_HOSTNAME => {
    :ip  => SALT_MASTER1_IP,
    :cpu => SALT_MASTER1_CPU,
    :mem => SALT_MASTER1_MEM
  },

  SALT_MINION1_HOSTNAME => {
    :ip  => SALT_MINION1_IP,
    :cpu => SALT_MINION1_CPU,
    :mem => SALT_MINION1_MEM
  },

  SALT_MINION2_HOSTNAME => {
    :ip  => SALT_MINION2_IP,
    :cpu => SALT_MINION2_CPU,
    :mem => SALT_MINION2_MEM
  },
}

# ------------------------- Inventory for all hosts -------------------------
INVENTORY = {
  # salt-masters
  "salt-master" => ["salt-master1", ],

  # salt-minions
  "salt-minion" => ["salt-minion1", "salt-minion2"],
}

if ENV['VAGRANT_BACKEND'] == nil
  BACKEND = "virtualbox"
else
  BACKEND = ENV['VAGRANT_BACKEND']
end
puts "Vagrant backend: #{BACKEND}"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  config.vm.box = BOX_NAME
  # config.vm.box_url = BOX_FILE_URL

  # Configure host
  config.hostmanager.enabled = true
  config.hostmanager.manage_host = true
  config.hostmanager.ignore_private_ip = false
  config.hostmanager.include_offline = true

  VMS.each do |vm_name, vm_settings|
    config.vm.define vm_name do |v|
      # Configure hostname
      v.vm.hostname = vm_name
      # Configure ip address
      v.vm.network :private_network, ip: vm_settings[:ip]
      # Configure vm system resources
      v.vm.provider "virtualbox" do |vb|
        vb.customize ["modifyvm", :id, "--cpus", vm_settings[:cpu]]
        vb.customize ["modifyvm", :id, "--memory", vm_settings[:mem]]
        vb.customize ["modifyvm", :id, "--cpuexecutioncap", "100"]
        vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
      end
    end
  end

  # provision ansible roles

  # static ip address work after vagrant provision
  # private interface down state bug: https://github.com/hashicorp/vagrant/issues/8166
  config.vm.provision "shell", inline: "sudo systemctl restart network"

  config.vm.provision "ansible" do |ansible|
    ansible.limit = "all"
    ansible.playbook = "vagrant-playbook.yml"
    ansible.host_key_checking = false
    # ansible.sudo = true
    ansible.verbose = ENV['ANSIBLE_VERBOSE'] ||= ANSIBLE_VERBOSE

    ansible.groups = INVENTORY
    ansible.extra_vars = {
        ansible_ssh_user: 'vagrant',
        ansible_ssh_args: '-o ForwardAgent=yes',
    }
    ansible.raw_ssh_args = ['-o UserKnownHostsFile=/dev/null']
  end
end
