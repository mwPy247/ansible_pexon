To setup Nginx via Ansible, I decided to use a virtual machine running on VirtualBox. To make this work some
network configuration has to be done first. In what follows I provide a quick walkthrough of how I set up 
the network and then I will explain the playbook and the configuration file.

# Ansible Network configuration
1. Install ansible on host
2. Verify installation: ansible —version
3. Install a Virtual machine (I used Virtual Box and the ubuntu-20.04.3-desktop-amd64 image)
4. Setup the network between host and vm to be able to ssh into the vm from the host (I use NAT with port forwarding for this)
    1. Set host-port to any unused port (I use 2281) 
    2. Set guest-port to 22 (default for ssh)
    3. Leave host-ip and guest-ip empty such that default values are used.
5. Boot the vm and enter the terminal
6. Configure ssh on the vm (is use a MacOS as a host, so SSH doesn’t have to be configured here): 
    1. Make sure that the system is up to date: sudo apt update
    2. Install the ssh server: sudo apt install ssh-server
    3. Verfiy installation: systemctl status sshd (service should be active now)
7. Verify ssh connection from host to vm: ssh <target_name> -p <port_number> 
    1. Fill in <target_name> and <port_number> as specified in the port forwarding rule for the vm. In my case <target_name> == nginx@127.0.0.1 and <port_number> == 2281
    2. Output should look like this:
    3. 
8. Setup key-based authentication:
    1. Run ssh-keygen -t rsa -b 4096 to generate a key
    2. Copy the public key to the vm: ssh-copy-id -i ~/.ssh/id_rsa.pub -p 2281 nginx@127.0.0.1 
    3. Verify that file has been copied on vm: cat ~/.ssh/authorized_keys (public key should now be printed to the terminal)
    4. Re-establish connection to vm and verify that login is possible without entering password of vm (except ssh passphrase if provided): ssh nginx@127.0.0.1 -p 2281 
9. Go to project directory and run to verify that connection can be established: ansible all -m ping 
10. Run command: ansible-playbook --ask-become-pass install.yml 
11. To access the server add another port forwarding rule (I use 8080 for localhost and 80 for the target vm, as this is the default port nginx listens to) 
12. Go to 127.0.0.1:8080 in your browser.

# Explanation of files:
- Hosts: the inventory file. Has only one entry for the vm we use. As the network is configured to forward ports from 127.0.01:2281 to the vm, we have to specify this information here.
- Specifies where ansible will find the inventory and private key files.
- index.html: the static web page we want to host via nginx.
- Playbook.yml
    - We can specify „hosts:all“ in the first line as we only have one machine.
    - „Become: true“ takes care of executing the playbook with root permissions
    - Set system time: this makes sure both system clocks (on target and on host) are synchronized as I encountered errors because of this.
    - Install nginx: update-cache makes sure to run apt update before installation
    - Copies our static webpage to the default nginx folder
