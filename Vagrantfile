# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
require 'yaml'
VAGRANTFILE_API_VERSION = "2"

base_dir = File.expand_path(File.dirname(__FILE__))
cluster = {
  "master1" => { :ip => "192.0.3.11",  :cpus => 1, :mem => 512 },
  #"master2" => { :ip => "192.0.3.12",  :cpus => 1, :mem => 512 },
  #"master3" => { :ip => "192.0.3.13",  :cpus => 1, :mem => 512 },
  "slave1"  => { :ip => "192.0.3.51", :cpus => 2, :mem => 512 },
  #"slave2"  => { :ip => "192.0.3.52", :cpus => 2, :mem => 512 },
  #"slave3"  => { :ip => "192.0.3.53", :cpus => 1, :mem => 512 },
  # "kafka-node1"   => { :ip => "100.0.20.101", :cpus => 1, :mem => 512 },
}

conf = YAML.load_file(File.join(base_dir, "digitalocean.yaml"))

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
    config.ssh.insert_key = false
    config.vm.box = "ubuntu/trusty64"
    config.vm.box_check_update = false


  cluster.each_with_index do |(hostname, info), index|
    config.vm.define hostname do |cfg|

      cfg.vm.provider :digital_ocean do |digital_ocean, override|
        override.ssh.private_key_path = conf["ssh_private_key_path"]
        override.vm.box = 'digital_ocean'
        override.vm.box_url = "https://github.com/smdahlen/vagrant-digitalocean/raw/master/box/digital_ocean.box"

        digital_ocean.token = conf["token"]
        digital_ocean.image = conf["image"]
        digital_ocean.region = conf["region"]
        digital_ocean.size = "512mb"
      end

      # provision nodes with ansible
      if index == cluster.size - 1
        cfg.vm.provision :ansible do |ansible|
          ansible.verbose = "v"
            ansible.limit = 'all_groups'
            ansible.groups = {
                "master_nodes" => ["master1", "master2", "master3"],
                "slave_nodes" => ["slave1", "slave2", "slave3"],
                "all_groups:children" => [ "master_nodes", "slave_nodes"]
            }
            ansible.playbook = "ansible/cluster.yml"
        end # end cfg.vm.provision
      end #end if

    end # end config

  end #end cluster

end #end vagrant
