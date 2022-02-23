To setup Nginx via Ansible, it is convenient to use a virtual machine running on VirtualBox. To make this work some
network configuration has to be done first. In what follows I provide a quick walkthrough of how to set up 
the network and then the configuration of Ansible and the structure and content of `playbook.yml` is explained.

# Ansible Network configuration
1. Install ansible on host
     => https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html 
3. Verify installation: `ansible —version`
4. Install a Virtual machine (I used Virtual Box and the ubuntu-20.04.3-desktop-amd64 image). I decided to call this machine `nginx`.
5. Setup the network between host and vm to be able to ssh into the vm from the host. I decided to use NAT with port forwarding for this. This is the configuration to be done inside VirtualBox:
    1. Go to network options of the vm and create a NAT adapter with the following port forwarding rule: 
        2. Set host-port to any unused port (I use 2281) 
        3. Set guest-port to 22 (default for ssh)
        4. Leave host-ip and guest-ip empty such that default values are used.
6. Boot the vm and enter the terminal
7. Configure ssh on the vm (I use MacOS locally as a host, so SSH doesn’t have to be configured here): 
    1. Make sure that the system is up to date: `sudo apt upgrade`
    2. Install the ssh server: `sudo apt install openssh-server`
    3. Verfiy installation: `systemctl status sshd` (The SSH-service should be active on the server now)
8. Verify ssh connection from host to vm: `ssh <target_name> -p <port_number>`. Fill in <target_name> and <port_number> as specified in the port forwarding rule for the vm. In my case <target_name> == nginx@127.0.0.1 and <port_number> == 2281. 
9. Setup key-based authentication:
    1. Run `ssh-keygen -t rsa -b 4096` to generate a key
    2. Copy the public key to the vm: `ssh-copy-id -i ~/.ssh/id_rsa.pub -p 2281 nginx@127.0.0.1`
    3. Verify that file has been copied on vm: `cat ~/.ssh/authorized_keys` (public key should now be printed to the terminal)
    4. Re-establish connection to vm and verify that login is possible without entering password of vm (except ssh passphrase if provided): `ssh nginx@127.0.0.1 -p 2281`
10. Go to project directory and verify that connection can be established: `ansible all -m ping`
11. Run command: `ansible-playbook --ask-become-pass playbook.yml`
12. To access the server add another port forwarding rule (I use 8080 for localhost and 80 for the target vm, as this is the default port nginx listens to) 
13. Visit 127.0.0.1:8080 in your local browser.

With these steps completed, I can access the Nginx server running on my remote vm within my local browser. Now the last step to be done is to serve a different .html-file. I do this with Ansible:

# Configuration of Ansible:
- *Hosts*: the inventory file. Has only one entry for the vm we use. As the network is configured to forward ports from 127.0.01:2281 to the vm, we have to specify this information here.
- *ansible.cfg* Specifies where ansible will find the inventory and private key files.
- *index.html* the static web page we want to host via nginx.
- *Playbook.yml* The most important file:
    - We can specify `hosts:all` in the first line as we only have one machine.
    - `Become: true` takes care of executing the playbook with root permissions
    - Set system time: this makes sure both system clocks (on target and on host) are synchronized (this turned out to be a problem on my setup).
    - Install nginx: update-cache makes sure to run `apt update` before installation
    - The last step copies our static webpage to the default nginx folder
 
We should now see `index.html` with the message "Hello from Ansible" on 127.0.0.1:8080 in the local browser.
