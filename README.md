# Puppet
How to's for installing and configuring puppet

## What is Puppet
Puppet is a configuration management tool for describing the state of the system. It capable of modeling your infrastucture. It is capable of adapting to mutiple types of OS or distro's.

## Install & Configuring Master
### Server/Master Node
#### Pre-requisites
Install puppet repo (make sure you are a root user)
> rpm -ivh http://yum.puppetlabs.com/puppet-release-el-7.noarch.rpm

#### Install Puppet Server application
> yum install -y puppetserver git

#### Reconfigure the memory allocation on java
You might need to drop down the default memory allocation from the 
> vi /etc/sysconfig/puppetserver

then going to JAVA_ARGS and change 2g in Xms and Xmx to say 512M or whatever that suits your requirement but should be around 50% of your VM's memory for systems to work smoothly

#### Starting Puppet Server
> systemctl start puppetserver

To start puppet on boot do
> systemctl enable puppetserver

#### Configure agent on the master
Set up the master server as on the agent section of /etc/puppetlabs/puppet/puppet.conf as follows
```
[agent]
server = yourhostname.com
```

Install r10k, its a tool for managing and deploying puppet environements and modules. To do that run 
> gem install r10k

Use the ruby installed on the puppet directory
>vi ~/.bash_profile
Add /opt/puppetlabs/puppet/bin to $PATH 
> source ~/.bash_profile

Or you can install latest ruby(2.3 or above) verion using rvm
```
 yum install -y gcc-c++ patch readline readline-devel zlib zlib-devel libyaml-devel libffi-devel openssl-devel make bzip2 autoconf automake libtool bison iconv-devel sqlite-devel
 curl -sSL https://rvm.io/mpapis.asc | gpg2 --import -
 curl -sSL https://rvm.io/pkuczynski.asc | gpg2 --import -
 curl -L get.rvm.io | bash -s stable
 source /etc/profile.d/rvm.sh
 rvm reload
 rvm install 2.6
```
#### Execute Puppet Agent
This will deploy the configurations defined on your environemt
> puppet agent -t 

If you dont see any proper output then it means that environment path is not set properly, set it using following config file:
> vi /etc/puppet/puppet.conf

``` 
[agent]
environmentpath=/etc/puppet/environments
```
To check environment path use this :
> puppet config print | grep environmentpath

Add these to your ~/.bash_aliases for easy jumping between directories.
> alias en='cd /etc/puppetlabs/code/environments'

> alias rk='cd /etc/puppetlabs/r10k'

#### Create r10k and environement based directories
> mkdir -p /var/cache/r10k
> mkdir -p /etc/puppetlabs/code/environments
> mkdir -p /etc/puppetlabs/r10k

#### Adding repo source for r10k
``` 
vi /etc/puppet/r10k/r10k.yaml
 ---
 :cachedir: '/var/cache/r10k'
 
 :sources:
         :my-org:
                 remote: 'https://your-git-repo'
                 basedir: '/etc/puppet/code/environments'
```
#### Deploy code 
From repo using r10k. Executing this will download the repo files to the code/environment 
> r10k deploy environement -p

### Signing Certificate 
puppet 5 certificate was signed or listed using
```
puppet cert list
puppet cert sign -a
```
Puppet 6 uses
```
puppetserver ca list
puppetserver ca sign --certname something
puppetserver ca sign -a # for signing all
```

### Slave/Agent Node
#### Install Puppet Agent application
> yum install -y puppet-agent

#### Evironment building
Following command will request puppet master to get a catelog of modifications.
If executed for the first time it will request for certificate signing request to the Master. You can sign request from the server using code mentioned on "Signing Certificate" section above
> puppet agent -t

## Creating resources
### File Resouces
To create/modify/check a file. The syntax starts with "file" keyword then file name. "ensure" keyword tells that its present(file/directory) or removed(absent) and content is for injecting content.
```
file { '/root/test':
    ensure => file,
    content => 'hello world!',
}
```
### Service Resource.
Describes as Service crond is ensure running and enabled to start on boot
```
service { 'httpd':
     ensure => running,
     enable => true,
}
```
site.pp is the default resource file which is been first read by the puppet master.
it should be present on your git repo, preferably on the manifest directory.

## Class:
Its a way of declaring an abstract of resources
#### Class Declaration syntax
```
class classname (
    $variable = 'test'
    ) {

}
```
#### Include Class primary syntax
```
node default {
  include classname 
}
```
#### Include class resource syntax
You can also include a class using the resource syntax. Here the class is taken as resource
```
node /^web/ {
    class { 'classname':
        variable => something,
    }
}
```

## Modules
These are predefined clases, you can find some at forge.puppet.com.
You can also create them yourself. Discussed later on this page.
To include a module you have to add this to the Puppetfile on the root of the r10k repo.
You have to include the dependencies of any existing Module.
```
vi Puppetfile
mod 'puppet/nginx'
mod 'puppet/nginx'
mod 'puppetlabs/stdlib'
mod 'puppetlabs/concat'
```
Your custom modules can go on the site folder 

### Profiles
It is a convention of declaring clases. As name suggest you should declare profile of node. For example 
>site/profile/manifests/web.pp

This class would contain you web related resources. Like for example you can include apache module or nginx module and setup your vhost etc.

### Roles
Its also a convention of declaring classes. As name suggests you should declare your server based declarations here. For example
>site/role/manifests/web_server.pp

This class is an abstract of web servers in your infratructure. So you have include the profile::web on it and instantiate it on site.pp.

### Facter 
Is a standalone tool to gather facts about a node. You can execute below line to get a list of facter variables set on master or agent nodes
```
facter # this gives all info
facter os.family # this a an object based query of os family
facter ipaddress # returns ip address
```
### Ordering Manifests execution
To order the files based on requirement we can use META parameters like before, require keyword. This can also act as a dependency system where you tell the config to check if previous item is ready or availble to execute existing resource.
