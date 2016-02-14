# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure(2) do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.


  # Create Mesos Master VM
  config.vm.define "master" do |master|

    master.vm.box = "centos/7"
    master.vm.hostname = "master"
    master.vm.network :private_network, ip: "192.168.101.2"

    # Create a forwarded port mapping which allows access to a specific port
    # within the machine from a port on the host machine. In the example below,
    # accessing "localhost:8080" will access port 80 on the guest machine.
    master.vm.network "forwarded_port", guest: 8080, host: 8080
    master.vm.network "forwarded_port", guest: 5050, host: 5050

    master.vm.provider :virtualbox do |vb|
      # Use VBoxManage to customize the VM. For example to change memory:
      vb.customize ["modifyvm", :id, "--memory", "1024", "--cpus", "2"]
      vb.gui = true
    end

    # Enable provisioning with a shell script. Additional provisioners such as
    # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
    # documentation for more information about their specific syntax and use.
   master.vm.provision "shell", inline: <<-SHELL

      # install epel
      sudo yum -y install epel-release
      
      # install tools
      sudo yum -y install net-tools moreutils docker golang wget
      
      # setup golang env

       # enforce hate on firewalld
      sudo systemctl stop firewalld.service
      sudo systemctl disable firewalld.service
      sudo yum erase -y firewalld

      sudo echo -e "127.0.0.1\tlocalhost\n::1\tlocalhost\n$(ifdata -pa eth1)\t$(hostname)" > /etc/hosts
      sudo cat /home/vagrant/sync/hosts >> /etc/hosts
   
      # Install Mesos repo and packages
      sudo rpm -Uvh http://repos.mesosphere.com/el/7/noarch/RPMS/mesosphere-el-repo-7-1.noarch.rpm
      sudo yum --enablerepo=mesosphere clean metadata
      sudo yum -y install mesos mesosphere-zookeeper
      sudo yum -y install marathon

      # Start Mesos Core Services
      sudo echo "1" > /var/lib/zookeeper/myid
      sudo systemctl enable zookeeper.service
      sudo systemctl enable mesos-master.service
      sudo systemctl enable marathon.service
      sudo service zookeeper start
      sudo service mesos-master start
      sudo service marathon start

      # Disable Mesos Slave Service on Masters
      sudo systemctl stop mesos-slave.service
      sudo systemctl disable mesos-slave.service
    SHELL
  end

  # Create Mesos Node1 VM
  config.vm.define "mnode1" do |mnode1|

    mnode1.vm.box = "centos/7"
    mnode1.vm.hostname = "mnode1"
    mnode1.vm.network :private_network, ip: "192.168.101.3"

    mnode1.vm.provider :virtualbox do |vb|
      # Use VBoxManage to customize the VM. For example to change memory:
      vb.customize ["modifyvm", :id, "--memory", "1024", "--cpus", "2"]
      vb.gui = true
    end

    # Enable provisioning with a shell script. Additional provisioners such as
    # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
    # documentation for more information about their specific syntax and use.
   mnode1.vm.provision "shell", inline: <<-SHELL

      # install epel
      sudo yum -y install epel-release
      sudo yum -y install net-tools moreutils docker

      # enforce hate on firewalld
      sudo systemctl stop firewalld.service
      sudo systemctl disable firewalld.service
      sudo yum erase -y firewalld

      # install mesos repo and packages
      sudo rpm -Uvh http://repos.mesosphere.com/el/7/noarch/RPMS/mesosphere-el-repo-7-1.noarch.rpm
      sudo yum --enablerepo=mesosphere clean metadata
      sudo yum -y install mesos 

      # setup zookeeper and mesos-slave items
      sudo echo "192.168.101.2 master" >> /etc/hosts
      sudo echo "zk://192.168.101.2:2181/mesos" > /etc/mesos/zk

      sudo echo "$(hostname)" > /etc/mesos-slave/hostname
      sudo echo "$(ifdata -pa eth1)" > /etc/mesos-slave/ip
      sudo echo "docker,mesos" > /etc/mesos-slave/containerizers
      
      # open ports for mesos
      sudo iptables -A INPUT -p tcp --match multiport --dports 31000:32000 -j ACCEPT

      sudo systemctl stop mesos-master.service
      sudo systemctl disable mesos-master.service


      sudo systemctl enable mesos-slave.service
      sudo systemctl enable docker.service
      sudo systemctl start mesos-slave.service
      sudo systemctl start docker.service

    SHELL
  end

  # Create Mesos Node2 VM
  config.vm.define "mnode2" do |mnode2|

    mnode2.vm.box = "centos/7"
    mnode2.vm.hostname = "mnode2"
    mnode2.vm.network :private_network, ip: "192.168.101.4"

    mnode2.vm.provider :virtualbox do |vb|
      # Use VBoxManage to customize the VM. For example to change memory:
      vb.customize ["modifyvm", :id, "--memory", "1024", "--cpus", "1"]
      vb.gui = true
    end

    # Enable provisioning with a shell script. Additional provisioners such as
    # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
    # documentation for more information about their specific syntax and use.
   mnode2.vm.provision "shell", inline: <<-SHELL

      # install epel
      sudo yum -y install epel-release
      sudo yum -y install net-tools moreutils docker

      # enforce hate on firewalld
      sudo systemctl stop firewalld.service
      sudo systemctl disable firewalld.service
      sudo yum erase -y firewalld

      # install mesos repo and packages
      sudo rpm -Uvh http://repos.mesosphere.com/el/7/noarch/RPMS/mesosphere-el-repo-7-1.noarch.rpm
      sudo yum --enablerepo=mesosphere clean metadata
      sudo yum -y install mesos 

      # setup zookeeper and mesos-slave items
      sudo echo "192.168.101.2 master" >> /etc/hosts
      sudo echo "zk://192.168.101.2:2181/mesos" > /etc/mesos/zk

      sudo echo "$(hostname)" > /etc/mesos-slave/hostname
      sudo echo "$(ifdata -pa eth1)" > /etc/mesos-slave/ip
      sudo echo "docker,mesos" > /etc/mesos-slave/containerizers
      
      # open ports for mesos
      sudo iptables -A INPUT -p tcp --match multiport --dports 31000:32000 -j ACCEPT

      sudo systemctl stop mesos-master.service
      sudo systemctl disable mesos-master.service


      sudo systemctl enable mesos-slave.service
      sudo systemctl enable docker.service
      sudo systemctl start mesos-slave.service
      sudo systemctl start docker.service

    SHELL
  end

  # Create Mesos Node3 VM
  config.vm.define "mnode3" do |mnode3|

    mnode3.vm.box = "centos/7"
    mnode3.vm.hostname = "mnode3"
    mnode3.vm.network :private_network, ip: "192.168.101.5"

    mnode3.vm.provider :virtualbox do |vb|
      # Use VBoxManage to customize the VM. For example to change memory:
      vb.customize ["modifyvm", :id, "--memory", "1024", "--cpus", "2"]
      vb.gui = true
    end

    # Enable provisioning with a shell script. Additional provisioners such as
    # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
    # documentation for more information about their specific syntax and use.
    mnode3.vm.provision "shell", inline: <<-SHELL

      # install epel
      sudo yum -y install epel-release
      sudo yum -y install net-tools moreutils docker

      # enforce hate on firewalld
      sudo systemctl stop firewalld.service
      sudo systemctl disable firewalld.service
      sudo yum erase -y firewalld

      # install mesos repo and packages
      sudo rpm -Uvh http://repos.mesosphere.com/el/7/noarch/RPMS/mesosphere-el-repo-7-1.noarch.rpm
      sudo yum --enablerepo=mesosphere clean metadata
      sudo yum -y install mesos 

      # setup zookeeper and mesos-slave items
      sudo echo "192.168.101.2 master" >> /etc/hosts
      sudo echo "zk://192.168.101.2:2181/mesos" > /etc/mesos/zk

      sudo echo "$(hostname)" > /etc/mesos-slave/hostname
      sudo echo "$(ifdata -pa eth1)" > /etc/mesos-slave/ip
      sudo echo "docker,mesos" > /etc/mesos-slave/containerizers
      
      # open ports for mesos
      sudo iptables -A INPUT -p tcp --match multiport --dports 31000:32000 -j ACCEPT

      sudo systemctl stop mesos-master.service
      sudo systemctl disable mesos-master.service


      sudo systemctl enable mesos-slave.service
      sudo systemctl enable docker.service
      sudo systemctl start mesos-slave.service
      sudo systemctl start docker.service

      SHELL
    end

  # Create Mesos Master VM
  config.vm.define "lb" do |lb|

    lb.vm.box = "centos/7"
    lb.vm.hostname = "lb"
    lb.vm.network :private_network, ip: "192.168.101.6"

    # Create a forwarded port mapping which allows access to a specific port
    # within the machine from a port on the host machine. In the example below,
    # accessing "localhost:8080" will access port 80 on the guest machine.
    lb.vm.network "forwarded_port", guest: 80, host: 80
    lb.vm.network "forwarded_port", guest: 443, host: 443

    lb.vm.provider :virtualbox do |vb|
      # Use VBoxManage to customize the VM. For example to change memory:
      vb.customize ["modifyvm", :id, "--memory", "1024", "--cpus", "2"]
      vb.gui = true
    end

    # Enable provisioning with a shell script. Additional provisioners such as
    # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
    # documentation for more information about their specific syntax and use.
   lb.vm.provision "shell", inline: <<-SHELL

      # install epel
      sudo yum -y install epel-release
      sudo yum -y install net-tools moreutils docker golang haproxy git
      # sudo  

       # enforce hate on firewalld
      sudo systemctl stop firewalld.service
      sudo systemctl disable firewalld.service
      sudo yum erase -y firewalld

      # make sure docker is running
      sudo systemctl enable docker.service
      sudo systemctl start docker.service

      # haproxy silliness
      sudo mkdir -p /run/haproxy

      sudo echo -e "127.0.0.1\tlocalhost\n::1\tlocalhost\n$(ifdata -pa eth1)\t$(hostname)" > /etc/hosts
      sudo cat /home/vagrant/sync/hosts >> /etc/hosts

    SHELL
  end
end