# Deploy OpenStack Mitaka using Packstack

## Step by step tutorial for deploying OpenStack using Packstack from RDO.


##Create the Virtual Machine
- Processors:
    - Number of processors: 2
    - Number of cores per processor 2
- Memory: 6GB RAM (Recommended)
- HDD - SATA - Minimum 60 GB (Recommended *Preallocated*)
- Network:
    - Network Adapter :  **NAT**
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
```

```bash
# Enable the extras
~ $ sudo yum install -y centos-release-openstack-mitaka

# Update the current packages:
~ $ sudo yum update -y

# Update the system
~ $ sudo yum upgrade

# Install the required tools
~ $ sudo yum install -y git vim openssh-server python-devel openstack-packstack

```




**IMPORTANT:** For non-English environment make sure your ```/etc/environment``` is populated as it follows:

```bash
~ $ sudo vim /etc/environment
```

```vim
LANG=en_US.utf-8
LC_ALL=en_US.utf-8
```

###Installing Openstack with Packstack

```bash
# Install Openstack

~ $ packstack --allinone

# After the install you can find the ```keystonerc_admin``` in your home folder

~ $ source ~/keystonerc_admin

# The password can be found in ```keystonerc_admin``` or by simply echo-ing it

~ $ echo $OS_PASSWORD
```


**IMPORTANT:** In case you wish to re-run packstack with a modified answerfile you can do the following:

####NOTE: by default ```$youranswerfile``` is called packstack-answer-$date-$time.txt

```bash
~ $ sudo packstack --answer-file=$youranswerfile
```

##For more details please consult the links below:

- https://www.rdoproject.org/install/quickstart/
- https://www.rdoproject.org/networking/neutron-with-existing-external-network/
- https://www.rdoproject.org/networking/floating-ip-range/
- https://www.rdoproject.org/install/adding-a-compute-node/

