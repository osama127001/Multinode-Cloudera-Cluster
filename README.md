# Multinode Cloudera Cluster on CentOS7s

This is a complete guide on how to setup a multi-node cloudera cluster on CentOS7 using VirtualBox. The cluster is developed to design and 
implemenet Bigdata Solutions and provide all the services that are required to serve the purpose.

**`Note:`** During the installation, if you are facing any issues, please visit the `Issues` part of the documentation as you might be facing the same issue that I faced. 

## 1. Downloading Required Stuff:
* [CentOS7 ISO Image](https://www.centos.org/download/)
* [VirtualBox](https://www.virtualbox.org/wiki/Downloads)

Now open virtual box and click on the **new** button in order to create a new VM (Virtual Machine).

## 2. Creating a VM using VirtualBox
* Click on the new button.
* Add the name of the VM as per your choice.
       1. Type: Linux
       2. Version: Red-Hat (64 bit)
* Assign memory according to the current Memory installed on your system. (4GB Recommended)
* Select **Create a virtual hard disk now** option
* Select **VDI (Virtualbox Disk Image)**
* Select **Dynamically Allocated** 
* Assign the storage according to the available storage on your system. (Atleast 8GB)
* Creating Adapters
       1. Goto File > Host Network Manager
       2. There are 3 buttons shown namely: **create, remove** and **properties**
       3. Create 2 Adapters and name them **vboxnet0** and **vboxnet1**
* Now go back to the VirtualBox and select the VM that is just created and click on the **settings** button.
* Goto **Adapter** tab.
       1. Under **Adapter2** tab, select the Enable Network Adapter checkbox and assign vboxnet0.
       2. Goto **Adapter3** tab, select the Enable Network Adapter checkbox and assign vboxnet1.
* Now goto **Storage** tab in settings and Select the **empty** disk. on the right side, add the CentOS7 ISO Image.
* Click **OK** and you are done.
* Now run the VM. 

After the creation of the VM is done, you can repeat step 2 in order to create as many nodes as you need for the cluster.
Note that the more nodes you create, the more resources will be consumed from your system. 
For a normal cluster, you need atleast **1 Master and 2 slaves**

## 3. Installation of CentOS7 on VM
   The Installation is relatively simple.
* Select the language.
* Add the appropriate date and time.
* Goto **Network and Hostname** and select all 3 adapters
* Click on **Begin Installation**
* After the Installation add the **Root Password**. (Creating a user is not that necessary)

After the Installation is done, the system will ask to reboot. after the reboot it will ask for the following credentials
* username: **root**
* password: **password that you just set in step 5**

after entering the following credentials, you are currently in the localhost.

## 4. Setup Static IP Address
   It is important to setup a static ip address in order to access the node on the network.

* To set a static IP address, use the following command to add configurations:

       vi /etc/sysconfig/network-scripts/ifcfg-**enp0s8**

Note that **enp0s8 is the name of the adapter in my machine**
* You will be prompted with the following contents of the file:

       TYPE=Ethernet
       BOOTPROTO=dhcp
       DEFROUTE=yes
       PEERDNS=yes
       PEERROUTES=yes
       IPV4_FAILURE_FATAL=no
       IPV6INIT=yes
       IPV6_AUTOCONF=yes
       IPV6_DEFROUTE=yes
       IPV6_PEERDNS=yes
       IPV6_PEERROUTES=yes
       IPV6_FAILURE_FATAL=no
       NAME=enp0s8
       UUID=c3fb64f0-6ff5-4294-9354-8e1279971d7e
       DEVICE=enp0s8
       ONBOOT=no

* make the following changes, set:

       BOOTPROTO=static
       ONBOOT=yes
       IPADDR=192.168.XXX.XXX
       NETMASK=255.255.255.0

* restert the network by executing the command:

       systemctl restart network

* Static IP will be set after the restart.


## 5. Check the availlable memory and storage
* to check the available memory of the system, use the following command:

       free -m

* to check the available storage, use the following command:

       df -h

## 5. Maximum number of open file requirements
* To check the current number of available number of open file descriptors, use the following command:

       ulimit -Sn
       ulimit -Hn

* To set the value of open file descriptors. (recommended: 10000)

       ulimit -n 10000

## 6. Changing the hostname of the machine
* By default, the hostname is: `localhost`
* To check the current running hostname, run the following command:

       hostname -f

* To check the details of hostname, run the command:

       hostnamectl

* To change the hostname of the system:

       hostnamectl set-hostname hostname

* reboot the system using the `reboot` command to make the changes.


## 7. Setting the hostname in the hostfile
* To view the hosts in the hosts file, enter the following command:

       vi /etc/hosts

* add the hosts in a fashion of adding an IP of a node and the hostname, the sample is given below:

       127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
       ::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
       192.168.XXX.XXX master
       192.168.YYY.XXX datanode1
       192.168.XXX.YYY datanode2

* in the above contents of the file, `master`, `datanode1` and `datanode2` are the names of the hosts.
* make sure to add these settings on each host of the cluster in order for the hosts to work.
* once these hosts are set, we can SSH any hosts using the hostname instead of the IP Address

       ssh master
       ssh datanode1
       ssh datanode2

## 8. Setup Password-less SSH
Password-less enables each node in the cluster to ssh any node without authentication or password. follow the steps given below in order to setup passwordless SSH
* Use the following command to generate a key. 

       ssh-keygen

* Press `enter` for all the default options provided to save the key. The command will generate 2 keys, a public key and a private key.

       Generating public/private rsa key pair.
       Enter file in which to save the key (/root/.ssh/id_rsa): 
       Created directory '/root/.ssh'.
       Enter passphrase (empty for no passphrase): 
       Enter same passphrase again: 
       Your identification has been saved in /root/.ssh/id_rsa.
       Your public key has been saved in /root/.ssh/id_rsa.pub.
       The key fingerprint is:
       97:0e:4c:a4:21:97:42:58:86:11:f1:74:e2:5a:82:cb root@localhost.localdomain
       The key's randomart image is:
       +--[ RSA 2048]----+
       |  +OB +..        |
       | .o=.=.+         |
       |. . +.. .        |
       |.. +   o   .     |
       |.E.     S o      |
       |         +       |
       |          .      |
       |                 |
       |                 |
       +-----------------+

* Now enter the following command to copy the generated key to the `authorized_keys` file:

       cat .ssh/id_rsa.pub >> authorized_keys

* now change the following permissions:

       chmod 700 ~/.ssh
       chmod 600 ~/authorized_keys

* Now use the following command to copy the key to the target host in order to set the password-less SSH:

       ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.XYX.XYX

* Note that in the command above, the target host can also be accessed using the hostname instead of using the IP.
* Enter the password of the target host. And now you can access 192.168.XYX.XYX from the current host
* repeat the above process in order to setup the Password-less SSH on other nodes as well.

## 9. Setup NTP on the server and the browser host
NTP ensures that the clocks of the nodes in the cluster are synchronized. follow the steps given below in order to set the NTP on each node:
* Install NTP using the command: 

       yum install -y ntp

* Enable NTP using the command; 

       systemctl enable ntpd

* following commands are used to start and restart the NTP service.

       systemctl start ntpd
       systemctl restart ntpd

* If you want to check the sync status, use the following command:

       ntpstat

## 10. Disabling Firewalls / Configuring IP tables

* execute the following 2 commands given below to disable the firewall

       systemctl disable firewalld
       service firewalld stop

## 11. Set the Network Config File

* Change the contents of the following file by using the command:

       vi /etc/sysconfig/network

* make the following changes in the file:

       NETWORKING=yes
       HOSTNAME=localhost

## 12. Disable SELinux and PackageKit and check umask value
It is important to disable SELinux for the ambari function to setup. 
* Use the following command to disable SELinux:

       setenforce 0

* to permenantly disable SELinux, make sure to set `SELINUX=disabled` in the following file to make sure that it is not enables after the system is rebooted:

       vi /etc/selinux/config

* For packagekit, set `enabled=0` in the following file:

       vi /etc/yum/pluginconf.d/refresh-packagekit.conf

* To check the current umask, run thhe following command:

       umask

* set the umask just fot the current login session: 

       umask 0022

* permenantly changing the umask for all the users: 

       echo umask 0022 >> /etc/profile
