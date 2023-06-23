# Ansible Playbooks

This folder contains ansible playbooks to help setup Theta and Zeta from scratch. Ansible is a useful tool to programmatically configure servers.

Ansible strives to be idempotent, meaning that no matter how many times you run the playbook you’ll end up with the same result.

Warning: if running a playbook against a machine that’s already setup, settings like the cta user password will be changed to that in the playbook.

## Testing
Best way to test these playbooks is with vagrant as a method to create virtual machines (VMs) locally.

### Prerequisites:
- Ansible (for actually running the playbooks)
- Vagrant (for creating / managing VMs for testing)
- libvirt / virtualbox (for actually running VMs)
- nfs-utils (a requirement of the Vagrant Debian image)

Move to temp folder
```bash
cd ./Testing
```

Grab debian image
```bash
vagrant box add debian/buster64
```

Create default config
```bash
vagrant init debian/buster64
```

Add / edit vagrant file to include relevant playbook
```
config.vm.provision "ansible" do |ansible|
        ansible.playbook = "zeta-playbook.yml"
    end
```

Create and start VM
```bash
vagrant up
```

Dial into the VM to have a play and test
```bash
vagrant ssh
```

## Execution / Installation
### Local prerequisites
- Ansible (for actually running the playbooks)

Install Debian (## Need to verify current version ##) on remote / target hardware with sane settings. Bare minimum requirements are ssh access and a user that belongs to sudo group.

(Warning: the non-root user created whilst following the Debian GUI installer is NOT automatically added to sudoers. To do so run this:
```bash
su -c 'gpasswd -a $USER sudo'  # Swap $USER for actual name```
)

Then it’s a case of use ansible to run the playbook with:
```bash
ansible-playbook -i 192.168.121.146, \  # Hosts to run on
 -u $USER_WITH_SUDO \   # User with sudo
 -k \               # Ask for SSH password
 -K \               # Ask for sudo password
 zeta-playbook.yml  # Playbook to run
```