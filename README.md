# Theta Setup

This is a run down of how to get a machine up and running as Theta.
Any device that needs to be set up us as Theta will need two NICs. There will also be a corresponding Ansible playbook.

## High level

Theta has several roles:
- DHCP and PXE boot server for device wiping network
- Repository for device wipe logs
- Host for Windows 10, 11 and Server VMs
- Host for CTA DB development VM
- (Proxy for CTA ShredOS queries to live DB)

### Wiping network

The wiping network has several elements:
- `isc-dhcp-server` is configured to run as a DHCP server on the NIC that has been chosen for the wiping network. It is also configured to point PXE booting devices at `menu.ipxe` (the PXE image menu).
- `tftp-hpa` is configured to serve the base images from `/srv/netboot` needed for PXE booting. (Looks like we could remove this as most devices support iPXE and HTTP.)
- `busybox` is used as a lightweight HTTP server to serve up `menu.ipxe` and the other files needed for devices to boot (.isos, kernels, initrd .imgs etc).
- `vsftpd` is used for FTP serving for receiving log files from ShredOS as well as serving the HardwardInfo scripts for the TEST version of ShredOS

### VM host

Hosts various guest systems via QEMU/KVM and libvirt:
- Windows 10 /11 for imaging and update serving
- Windows Server for file sharing and Windows Deployment Services (PXE booting Windows installs)
- Debian images for CTA app development.

## Process

1. Install Debian on the machine using sane settings. (British English etc).
    1. Set root password: $HOSTNAME+Creativity24%
    2. Best to create personal login
    3. Make sure to select SSH server

2. On log in create additional users:
    1. Generic CTA user:
        `useradd -m -p $DefaultPasswordStem+$Year cta`
    2. netboot-log user for ShredOS to use to stash wiping logs
        `useradd -d /srv/netboot/log -M -p $NetbootLogPassword netboot-log`

3. Add a user to sudoers
    `usermod -aG sudo $user`

4. Install general tools
    1. `sudo apt install zsh tree htop git curl`
    2. sublime text !!!
    3. zerotier     !!!
    4. ?atuin?      ???

5. Disable power saving to make sure device doesn't suspend by creating `/etc/systemd/sleep.conf.d/nosuspend.conf` and adding the following to the file:
    ```
    [Sleep]
    AllowSuspend=no
    AllowHibernation=no
    AllowSuspendThenHibernate=no
    AllowHybridSleep=no
    ```

6. Sort out PXE booting set up
    1. Install packages:
        `sudo apt install isc-dhcp-server tftpd-hpa`
    2. Set up DHCP
        1. Get DHCP confs
            /etc/dhcp/dhcpd.conf
            /etc/default/isc-dhcp-server
        2. Config secondary ethernet port for wiping LAN by appending the following to /etc/interfaces
            ```
            # Wiping LAN conf
            allow-hotplug $DEV
            iface $DEV inet static
                address 10.0.0.1
                netmask 255.255.255.0
            ```
        3. Start dhcpd server:
            `sudo systemctl enable --now isc-dhcp-server`
    3. Set up TFTP
        1. Get TFPT conf
            /etc/default/tftpd-hpa
        2. Start TFTP server
            `sudo systemctl enable --now tftp-hpa`
    4. Fetch files needed for PXE booting
        1. iPXE files
            `wget http://boot.ipxe.org/undionly.kpxe`
            `wget http://boot.ipxe.org/ipxe.efi`
            `sudo cp ./undionly.kpxe /srv/netboot/`
            `sudo cp ./ipxe.efi /srv/netboot/`
        2. Fetch CTA PXE menu
            `wget .... /srv/netboot/menu.ipxe`
    5. Set up HTTP server
        1. Fetch busybox httpd service file
            `wget .... /etc/systemd/system/BusyBoxHTTP.service`
        2. Start and enable Busybox http service
            `sudo systemctl daemon-reload`
            `sudo systemctl enable --now BusyBoxHTTP.service
    6. Populate files needed for PXE booting various OSes.
        1. ShredOS
            TODO (File structure, mounting, fstab, point to repo, build image?)
        2. Blancco 
            TODO (-----------''------------------, customise ISOs?)
        3. PartedMagic
            TODO

7. ShredOS Log repository. (Just bog standard folders for the moment.)
    1. Create required folder structure
        `mkdir -p /srv/netboot/log/shredos`
        `mkdir -p /srv/netboot/log/test-shredos`
    2. Change ownership so `netboot-log` user has access
        `chown -R netboot-log:netboot-log /srv/netboot/log`

8. Hardware Scripts (for TEST ShredOS)
    1. Create required folder structure
        `mkdir -p /srv/netboot/log/scripts`
    2. Fetch scripts from repo
        `cd ~; git clone https://github.com/CommunityTechaid/HardwardInfo.git`
    3. Copy scripts from repo to folder
        `sudo cp ./HardwardInfo/scripts/* /srv/netboot/scripts/`
    4. Change ownership to make sure `netboot-log` user has access
        `chown -R netboot-log:netboot-log /srv/netboot/log`

9. VSFTPD
    1. Install vsftpd
        `sudo apt install vsftpd`
    2. Get conf
        /etc/vsftpd/conf
    3. Change folder permissons to avoid issue
        `sudo chmod a-w /srv/netboot/log`
    4. Enable and start service
        `sudo systemctl enable --now vsftpd`

10. Automagic scripts
    1. Install prerequisites
        `sudo apt install pip`
    2. Fetch scripts from repo
        `cd ~; git clone https://github.com/CommunityTechaid/automagic-device-update.git`
    3. Create, and activate virtual environment
        `python -m venv venv`
        `source ./venv/bin/activate`
    4. Install requirements
        `pip install -r requirements.txt`


10. LibVirt & Virtual Machines  
    TODO

