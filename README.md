# Deploy OpenStack Mitaka using Packstack

## Step by step tutorial for deploying OpenStack using Packstack from RDO.


##Create the Virtual Machine
- Processors:
    - Number of processors: 2
    - Number of cores per processor 2
- Memory: 6GB RAM (Recommended)
- HDD - SATA - Minimum 60 GB (Recommended *Preallocated*)
- Network:
    - Network Adapter :  **Bridged(Replicate network connection state)**
- Operating system - **CentOS 7** (Recommended) 

Note: The Hypervisor used for this example is **VMWare Workstation 11**

**IMPORTANT:** Do not forget to enable ```Virtualize Intel VT-x/EPT or AMD-V/RVI``` in Processors settings.

##Setting up the system

###Prepare the network
```bash
~ $ sudo systemctl disable firewalld
~ $ sudo systemctl stop firewalld
~ $ sudo systemctl disable NetworkManager
~ $ sudo systemctl stop NetworkManager
~ $ sudo systemctl enable network
~ $ sudo systemctl start network
~ $ sudo yum remove NetworkManager
~ $ sudo systemctl mask firewalld
~ $ sudo systemctl disable firewalld
~ $ sudo systemctl mask NetworkManager

```

```bash
# Enable the extras
~ $ sudo yum install -y centos-release-openstack-mitaka

# Update the current packages:
~ $ sudo yum update -y

# Update the system
~ $ sudo yum upgrade

# Install the required tools
~ $ sudo yum install -y git vim openssh-server python-devel openstack-packstack deltarpm yum-utils yum-cron net-tools qemu-kvm qemu-kvm-tools wget

# Clean-up old kernels

~ $ sudo package-cleanup --oldkernels --count 2
```


**IMPORTANT:** For non-English environment make sure your ```/etc/environment``` is populated as it follows:

```bash
~ $ sudo vim /etc/environment
```

```vim
LANG=en_US.utf-8
LC_ALL=en_US.utf-8
```

## Configure the hosts and network

```bash
# Modify the hosts so that it resembles the snippet below
~ $ sudo vim /etc/hosts
```

```vim
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.133.141 localhost work-argus-ci-test # The hostname IP should be set accordingly

```


```bash
# Modify the network config, the name might differ

~ $ sudo vim /etc/sysconfig/network-scripts/ifcfg-enp0s25
```

```vim
TYPE=Ethernet
BOOTPROTO=static # Set it to static
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
NAME="device-name"
UUID="the-uid"
DEVICE="device-name"
ONBOOT=yes
PEERDNS=yes
PEERROUTES=yes
IPV6_PEERDNS=yes
IPV6_PEERROUTES=yes

IPADDR=192.168.133.141 # Set the IP Address that should be used by the mahcine
NETMASK=255.255.255.0
GATEWAY=192.168.133.1
DNS1=192.168.133.1
DNS2=8.8.8.8

```
## Configure the network_init
```bash
# Restart the network

~ $ sudo service network restart

# Get the network setup script

~ $ wget https://raw.githubusercontent.com/stefan-caraiman/packstack-tutorial/master/script.sh
~ $ chmod +x network_init.sh

# Also don't forget to modify the values from the script

# Next we need to have the network_init run at startup
# Add "bash /home/work/network_init.sh" to the end of the following file

~ $ sudo vim /etc/rc.local
~ $ sudo chmod +x /etc/rc.local

# Now reboot the system

~ $ sudo reboot
```

###Installing Openstack with Packstack

```bash
# Install Openstack using our customized answer file.
# Also do not forget to modify the HOST IP fields found in the file.

~ $ wget https://raw.githubusercontent.com/stefan-caraiman/packstack-tutorial/master/packstack-answers.txt

~ $ sudo packstack --answer-file=packstack.txt

# After the install you can find the ```keystonerc_admin``` in your home folder

~ $ sudo su
~ $ source ~/keystonerc_admin

# The password can be found in ```keystonerc_admin``` or by simply echo-ing it

~ $ echo $OS_PASSWORD

# Add the port for the br-ex

~ $ sudo ovs-vsctl add-port br-ex "the-eno-name"
```

## Configure the network with neutron

```bash

# Delete the default public network

~ $ neutron net-delete public

# Create a new public network

~ $ neutron net-create --shared --router:external public

# Configure the newly created public network

~ $ neutron subnet-create public 192.168.133.0/24 --name public_subnet --enable-dhcp=False --allocation-pool start=192.168.133.141,end=192.168.133.151 --gateway 192.168.133.1

# Create a new router and set the gateway for it

~ $ neutron router-create router
~ $ neutron router-gateway-set $router_id $public_network_id

# Add the private subnet to the routers interfaces

~ $ neutron router-interface-add router private_subnet

# Update the private_subnet DNS nameserver
# Run ```neutron subnet-list``` to see the subnets IDs.

~ $ neutron subnet-update --dns-nameserver 8.8.8.8 "private_subnet_id"
```

## Install tempest as a service

```bash

~ $ sudo yum install openstack-tempest
# Install pip
~ $ pip install virtualenv 
# Create an virtualenv
~ $ sudo virtualenv /usr/share/openstack-tempest-10.0.0/.venv
~ $ source /usr/share/openstack-tempest-10.0.0/.venv/bin/activate
# Install the dependecies under the virtualenv
~ $ pip install -r /usr/share/openstack-tempest-kilo/requirements.txt
~ $ pip install -r /usr/share/openstack-tempest-kilo/test-requirements.txt

```

## Other details
**IMPORTANT:** In case you wish to re-run packstack with a updated answerfile you can simply run the following:

####NOTE: by default ```$youranswerfile``` is called packstack-answer-$date-$time.txt

```bash
~ $ sudo packstack --answer-file=$youranswerfile
```

##For more details please consult the links below:

- https://www.rdoproject.org/install/quickstart/
- https://www.rdoproject.org/networking/neutron-with-existing-external-network/
- https://www.rdoproject.org/networking/floating-ip-range/
- https://www.rdoproject.org/install/adding-a-compute-node/

