# -*- mode: ruby -*-
# vi: set ft=ruby :

require_relative './script/authorize_key'

# NOTE: currently using the same OS for all boxen
OS="centos" # "debian" || "centos"

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure(2) do |config|
  package=""
  if OS=="debian"
    config.vm.box = "debian/jessie64"
    package="_apt"
  elsif OS=="centos"
    config.vm.box = "centos/7"
    package="_yum"
  else
    puts "you must set the OS variable to a valid value before continuing"
    exit
  end

  {
    'ansiblebox' => '10.11.12.101',
    #'db'          => '10.11.12.102',
    #'solr'        => '10.11.12.103'
  }.each do |short_name, ip|
    config.vm.define short_name do |host|
      host.vm.network 'private_network', ip: ip
      host.vm.hostname = "#{short_name}.boxen.dev"
      # presumes installation of https://github.com/cogitatio/vagrant-hostsupdater on host
      host.hostsupdater.aliases = ["#{short_name}"]

      host.vm.provider "virtualbox" do |vb|
        vb.name = "#{short_name}.boxen.dev"
      end

      # # provision authorized ssh key
      # # NOTE: the problem with this approach is that *sometimes*
      # # the file provisioner hasn't done its job by the time the prereqs script runs.
      # # when this happens, the key is not yet in its expected place to be moved over
      # # instead, calling the authorize_key.rb script modified from
      # # http://hakunin.com/six-ansible-practices
      # # this script just calls a file provisioner anyway,
      # # I believe that doing it in the same script that uses the file
      # # ensures that the provisioning step completes before the next tries to use it...
      # host.vm.provision "file", source: "public_ssh_key/authorized_key.pub", destination: ".ssh/authorized_key.pub"

      # do minimal provisioning (in order to do further work with Ansible)
      host.vm.provision "prerequisites", type: "shell", path: "script/prereqs#{package}.sh"

      # add authorized key to user created by the prereqs script
      authorize_key host, "deploy", "~/.ssh/personal_dev.pub"
    end
  end

  # include a playbook in provisioning
  # vm (re)defined here must match a shortname above
  config.vm.define "ansiblebox" do |ansiblebox|
    ansiblebox.vm.provision "ansible" do |ansible|
      ansible.playbook = "playbook#{package}.yml"
    end
  end

  # REM:
  # test: ansible all -m ping
  # run playbooks post-provisioning: ansible-playbook playbook.yml -i hosts
end
