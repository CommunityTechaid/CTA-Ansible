# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  config.vm.define "TEST-Theta"

  config.vm.box = "debian/bookworm64"

  config.vm.hostname = "TEST-Theta"

  config.vm.network "private_network", ip: "10.0.0.1"

  config.vm.post_up_message = "Theta has been provisioned"

  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "Theta.yml"
  end

end
