# Kubernetes Setup Using Ansible and Vagrant

## Overview
The aim of this project is having a one line command for lifting a kubernetes cluster in your local machine fully provisioned. This is also easily extensible for installing your own stuff using Ansible.

forked from https://github.com/borjatur/kubernetes-using-ansible-vagrant

## Few changes that I made to run it on my Ubuntu-20.04 machine
1.  changed the bento/ubntu version to 20.04 => IMAGE_NAME = "bento/ubuntu-20.04"
 why?
 there was a bug due to this ticket https://www.virtualbox.org/ticket/16670:
 ```
 Vagrant was unable to mount VirtualBox shared folders. This is usually
because the filesystem "vboxsf" is not available. This filesystem is
made available via the VirtualBox Guest Additions and kernel module.
Please verify that these guest additions are properly installed in the
guest. This is not a bug in Vagrant and is usually caused by a faulty
Vagrant box. For context, the command attempted was:

mount -t vboxsf -o uid=1000,gid=1000 vagrant /vagrant

The error output from the command was:

/sbin/mount.vboxsf: mounting failed with the error: No such device

 ```
 so update of vagrant, vb and the nodes image for me was the best fix and no got and manually change ln or install plugins
 
 2. TASK [./roles/worker-node : Copy the join command to server location] **********
  rises en error:
  ```
  An exception occurred during task execution. 
To see the full traceback, use -vvv. 
The error was: 
If you are using a module and expect the file to exist on the remote,
see the remote_src option
fatal: [node-1]: FAILED! => {"changed": false, "msg": 
"Could not find or access 
'./output/join-command'\n
Searched in:\n\t
/home/dev/Documents/kubernetes-using-ansible-vagrant/kubernetes-setup/roles/worker-node/files/./output/join-command\n\t
/home/dev/Documents/kubernetes-using-ansible-vagrant/kubernetes-setup/roles/worker-node/./output/join-command\n\t
/home/dev/Documents/kubernetes-using-ansible-vagrant/kubernetes-setup/roles/worker-node/tasks/files/./output/join-command\n\t
/home/dev/Documents/kubernetes-using-ansible-vagrant/kubernetes-setup/roles/worker-node/tasks/./output/join-command\n\t
/home/dev/Documents/kubernetes-using-ansible-vagrant/kubernetes-setup/files/./output/join-command\n\t
/home/dev/Documents/kubernetes-using-ansible-vagrant/kubernetes-setup/./output/join-command on the Ansible Controller.\n
If you are using a module and expect the file to exist on the remote, see the remote_src option"

  ```
  after littel research, in file master-node/tasks/main.yml:
  - name: Copy join command to local file
  copy: content="{{ join_command.stdout_lines[0] }}" dest="./output/join-command"
  delegate_to: 127.0.0.1
  become: false
  
  Copy method not created the 'join-command file'
  and from ansible docks:
  If C(directory), all immediate subdirectories will be created if they do not exist.\nIf C(file), the file will NOT be created if it does not exist, see the M(copy) or M(template) module if you want that behavior.  If C(absent), directories will be recursively deleted, and files will be removed.\nIf C(touch), an empty file will be created if the C(path) does not exist, while an existing file or directory will receive updated file access and modification times (similar to the way C(touch) works from the command line).
  so i crated that file manually
  
  
  
  full output:
  dev@dev:~/Documents/kubernetes-using-ansible-vagrant$ vagrant up
Bringing machine 'k8s-master' up with 'virtualbox' provider...
Bringing machine 'node-1' up with 'virtualbox' provider...
==> k8s-master: Importing base box 'bento/ubuntu-20.04'...
==> k8s-master: Matching MAC address for NAT networking...
==> k8s-master: Checking if box 'bento/ubuntu-20.04' version '202107.28.0' is up to date...
==> k8s-master: Setting the name of the VM: kubernetes-using-ansible-vagrant_k8s-master_1631804309724_94388
==> k8s-master: Clearing any previously set network interfaces...
==> k8s-master: Preparing network interfaces based on configuration...
    k8s-master: Adapter 1: nat
    k8s-master: Adapter 2: hostonly
==> k8s-master: Forwarding ports...
    k8s-master: 22 (guest) => 2222 (host) (adapter 1)
==> k8s-master: Running 'pre-boot' VM customizations...
==> k8s-master: Booting VM...
==> k8s-master: Waiting for machine to boot. This may take a few minutes...
    k8s-master: SSH address: 127.0.0.1:2222
    k8s-master: SSH username: vagrant
    k8s-master: SSH auth method: private key
==> k8s-master: Machine booted and ready!
==> k8s-master: Checking for guest additions in VM...
==> k8s-master: Setting hostname...
==> k8s-master: Configuring and enabling network interfaces...
==> k8s-master: Mounting shared folders...
    k8s-master: /vagrant => /home/dev/Documents/kubernetes-using-ansible-vagrant
==> k8s-master: Running provisioner: ansible...
    k8s-master: Running ansible-playbook...

PLAY [all] *********************************************************************

TASK [Gathering Facts] *********************************************************
ok: [k8s-master]

TASK [./roles/common : Install packages that allow apt to be used over HTTPS] ***
changed: [k8s-master]

TASK [./roles/common : Add an apt signing key for Docker] **********************
changed: [k8s-master]

TASK [./roles/common : Add apt repository for stable version] ******************
changed: [k8s-master]

TASK [./roles/common : Install docker and its dependecies] *********************
changed: [k8s-master]

TASK [./roles/common : Docker daemon config] ***********************************
changed: [k8s-master]

TASK [./roles/common : Restar docker] ******************************************
changed: [k8s-master]

TASK [./roles/common : Add vagrant user to docker group] ***********************
changed: [k8s-master]

TASK [./roles/common : Disable swap] *******************************************
changed: [k8s-master]

TASK [./roles/common : Add an apt signing key for Kubernetes] ******************
changed: [k8s-master]

TASK [./roles/common : Adding apt repository for Kubernetes] *******************
changed: [k8s-master]

TASK [./roles/common : Install Kubernetes binaries] ****************************
changed: [k8s-master]

TASK [./roles/common : Configure node ip] **************************************
changed: [k8s-master]

TASK [./roles/master-node : Config kubeadm images] *****************************
changed: [k8s-master]

TASK [./roles/master-node : Initialize the Kubernetes cluster using kubeadm] ***
changed: [k8s-master]

TASK [./roles/master-node : Setup kubeconfig for vagrant user] *****************
changed: [k8s-master] => (item=mkdir -p /home/vagrant/.kube)
changed: [k8s-master] => (item=cp -i /etc/kubernetes/admin.conf /home/vagrant/.kube/config)
changed: [k8s-master] => (item=chown vagrant:vagrant /home/vagrant/.kube/config)
[WARNING]: Consider using the file module with state=directory rather than
running 'mkdir'.  If you need to use command because file is insufficient you
can add 'warn: false' to this command task or set 'command_warnings=False' in
ansible.cfg to get rid of this message.
[WARNING]: Consider using the file module with owner rather than running
'chown'.  If you need to use command because file is insufficient you can add
'warn: false' to this command task or set 'command_warnings=False' in
ansible.cfg to get rid of this message.

TASK [./roles/master-node : Install calico pod network] ************************
changed: [k8s-master]

TASK [./roles/master-node : Generate join command] *****************************
changed: [k8s-master]

TASK [./roles/master-node : Copy join command to local file] *******************
changed: [k8s-master -> 127.0.0.1]

TASK [./roles/master-node : Copy kubeconfig to local file] *********************
changed: [k8s-master]

RUNNING HANDLER [./roles/master-node : docker status] **************************
ok: [k8s-master]

PLAY RECAP *********************************************************************
k8s-master                 : ok=21   changed=19   unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

==> node-1: Importing base box 'bento/ubuntu-20.04'...
==> node-1: Matching MAC address for NAT networking...
==> node-1: Checking if box 'bento/ubuntu-20.04' version '202107.28.0' is up to date...
==> node-1: Setting the name of the VM: kubernetes-using-ansible-vagrant_node-1_1631804489758_55655
==> node-1: Fixed port collision for 22 => 2222. Now on port 2200.
==> node-1: Clearing any previously set network interfaces...
==> node-1: Preparing network interfaces based on configuration...
    node-1: Adapter 1: nat
    node-1: Adapter 2: hostonly
==> node-1: Forwarding ports...
    node-1: 22 (guest) => 2200 (host) (adapter 1)
==> node-1: Running 'pre-boot' VM customizations...
==> node-1: Booting VM...
==> node-1: Waiting for machine to boot. This may take a few minutes...
    node-1: SSH address: 127.0.0.1:2200
    node-1: SSH username: vagrant
    node-1: SSH auth method: private key
==> node-1: Machine booted and ready!
==> node-1: Checking for guest additions in VM...
==> node-1: Setting hostname...
==> node-1: Configuring and enabling network interfaces...
==> node-1: Mounting shared folders...
    node-1: /vagrant => /home/dev/Documents/kubernetes-using-ansible-vagrant
==> node-1: Running provisioner: ansible...
    node-1: Running ansible-playbook...

PLAY [all] *********************************************************************

TASK [Gathering Facts] *********************************************************
ok: [node-1]

TASK [./roles/common : Install packages that allow apt to be used over HTTPS] ***
changed: [node-1]

TASK [./roles/common : Add an apt signing key for Docker] **********************
changed: [node-1]

TASK [./roles/common : Add apt repository for stable version] ******************
changed: [node-1]

TASK [./roles/common : Install docker and its dependecies] *********************
changed: [node-1]

TASK [./roles/common : Docker daemon config] ***********************************
changed: [node-1]

TASK [./roles/common : Restar docker] ******************************************
changed: [node-1]

TASK [./roles/common : Add vagrant user to docker group] ***********************
changed: [node-1]

TASK [./roles/common : Disable swap] *******************************************
changed: [node-1]

TASK [./roles/common : Add an apt signing key for Kubernetes] ******************
changed: [node-1]

TASK [./roles/common : Adding apt repository for Kubernetes] *******************
changed: [node-1]

TASK [./roles/common : Install Kubernetes binaries] ****************************
changed: [node-1]

TASK [./roles/common : Configure node ip] **************************************
changed: [node-1]

TASK [./roles/worker-node : Copy the join command to server location] **********
changed: [node-1]

TASK [./roles/worker-node : Join the node to cluster] **************************
changed: [node-1]

RUNNING HANDLER [./roles/worker-node : docker status] **************************
ok: [node-1]

PLAY RECAP *********************************************************************
node-1                     : ok=16   changed=14   unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

