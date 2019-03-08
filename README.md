# puppet
How to's for installing and configuring puppet

## Install & Configuring Master

### Pre-requisites
Install puppet repo (make sure you are a root user)
> rpm -ivh http://yum.puppetlabs.com/puppetlabs-release-el-7.noarch.rpm

### Install Puppet Server application
> yum install -y puppetserver

### reconfigure the memory allocation on java
You might need to drop down the default memory allocation from the 
> /etc/sysconfif/puppetserver

then going to JAVA_ARGS and change 2g to say 512M ,etc around 50% of your VM's memory for systems to work smoothly

### Starting Puppet Server
> systemctl start puppetserver

to start puppet on boot do
> systemctl enable puppetserver

### configure agent on the master
Set up the master server as on the agent section of /etc/puppet/puppet.conf as follows
> \[agent\]

> server = yourhostname.com

Install r10k, its a tool for managing and deploying puppet environements and modules. To do that run 
> gem install r10k
You have to install latest ruby(2.3 or above) verion or use the ruby installed on the puppet direcotry
