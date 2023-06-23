# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "debian/buster64"
  # Provision conf for Ansible
  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "zeta-playbook.yml"
  end
end
