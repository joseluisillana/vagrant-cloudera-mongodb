$master_script = <<SCRIPT
#!/bin/bash

apt-get install curl -y
REPOCM=${REPOCM:-cm5}
CM_REPO_HOST=${CM_REPO_HOST:-archive.cloudera.com}
CM_MAJOR_VERSION=$(echo $REPOCM | sed -e 's/cm\\([0-9]\\).*/\\1/')
CM_VERSION=$(echo $REPOCM | sed -e 's/cm\\([0-9][0-9]*\\)/\\1/')
OS_CODENAME=$(lsb_release -sc)
OS_DISTID=$(lsb_release -si | tr '[A-Z]' '[a-z]')
if [ $CM_MAJOR_VERSION -ge 4 ]; then
  cat > /etc/apt/sources.list.d/cloudera-$REPOCM.list <<EOF
deb [arch=amd64] http://$CM_REPO_HOST/cm$CM_MAJOR_VERSION/$OS_DISTID/$OS_CODENAME/amd64/cm $OS_CODENAME-$REPOCM contrib
deb-src http://$CM_REPO_HOST/cm$CM_MAJOR_VERSION/$OS_DISTID/$OS_CODENAME/amd64/cm $OS_CODENAME-$REPOCM contrib
EOF
curl -s http://$CM_REPO_HOST/cm$CM_MAJOR_VERSION/$OS_DISTID/$OS_CODENAME/amd64/cm/archive.key > key
apt-key add key
rm key
fi
apt-get update
export DEBIAN_FRONTEND=noninteractive
apt-get -q -y --force-yes install oracle-j2sdk1.7 cloudera-manager-server-db cloudera-manager-server cloudera-manager-daemons
sudo sysctl -w vm.swappiness=0
service cloudera-scm-server-db initdb
service cloudera-scm-server-db start
service cloudera-scm-server start
SCRIPT

$hosts_script = <<SCRIPT
cat > /etc/hosts <<EOF
127.0.0.1       localhost

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

EOF
SCRIPT

Vagrant.configure("2") do |config|

  # Define base image
  config.vm.box = "precise64"
  config.vm.box_url = "http://files.vagrantup.com/precise64.box"
  config.vm.synced_folder "#{ENV['HOME']}/virtual-hadoop-cluster/reporting/deployments/", "#{ENV['HOME']}/virtual-hadoop-cluster/reporting/data", :mount_options => ["dmode=777","fmode=777"]

  # Manage /etc/hosts on host and VMs
  config.hostmanager.enabled = false
  config.hostmanager.manage_host = true
  config.hostmanager.include_offline = true
  config.hostmanager.ignore_private_ip = false


  config.vm.define :master do |master|
    master.vm.provider :virtualbox do |v|
      v.name = "vm-cluster-node1"
      v.customize ["modifyvm", :id, "--memory", "5096"]
    end
    master.vm.network :private_network, ip: "10.211.55.100"
    master.vm.hostname = "vm-cluster-node1"
    master.vm.provision :shell, :inline => $hosts_script
    master.vm.provision :hostmanager
    master.vm.provision :shell, :inline => $master_script
  end



#  config.vm.define :slave1 do |slave1|
#  slave1.vm.box = "precise64"
#  slave1.vm.provider :virtualbox do |v|
#    v.name = "vm-cluster-node2"
#    v.customize ["modifyvm", :id, "--memory", "2048"]
#    end
#    slave1.vm.network :private_network, ip: "10.211.55.101"
#    slave1.vm.hostname = "vm-cluster-node2"
#    slave1.vm.provision :shell, :inline => $hosts_script
#    slave1.vm.provision :hostmanager
#  end

#  config.vm.define "appserver" do |app|
#    app.vm.box = "chef/fedora-20"
#    app.vm.network "forwarded_port", guest: 8080, host: 8080
#    app.vm.provider :virtualbox do |v|
#      v.name = "vm-appserver"
#      v.customize ["modifyvm", :id, "--memory", "1024"]
#    end    
#    app.vm.network :private_network, ip: "10.211.55.112"
#    app.vm.hostname = "vm-appserver"
#    app.vm.synced_folder "./webapps", "/var/lib/tomcat7/webapps", create:true, owner: "root", group: "root", mount_options: ["dmode=777,fmode=666"]
#    app.vm.synced_folder "./conf", "/etc/tomcat", create:true, owner: "root", group: "root", mount_options: ["dmode=777,fmode=666"]
#    app.vm.synced_folder "./log", "/var/log/tomcat", create:true, owner: "root", group: "root", mount_options: ["dmode=777,fmode=666"]
#    app.vm.provision "shell" do |s|
#       s.inline = "sudo apt-get update; sudo apt-get -y install puppet; sudo apt-get -y install tomcat7"
#    end
#  end


  config.vm.define "mongo-node-master" do |mongonode|
   mongonode.vm.box = "precise64"
   mongonode.vm.provider :virtualbox do |v|
      v.name = "vm-cluster-mongomaster"
      v.customize ["modifyvm", :id, "--memory", "1024"]
    end
    mongonode.vm.network :private_network, ip: "10.211.55.111"
    mongonode.vm.hostname = "vm-cluster-mongomaster"
    mongonode.vm.provision :hostmanager
    mongonode.vm.provision :chef_solo do |chef|
      chef.add_recipe "mongodb-debs"
    end
    mongonode.vm.provision :shell, :inline => "sudo apt-get install -y build-essential --no-install-recommends"
     mongonode.vm.provision :shell, :inline => "sudo apt-get install -y ruby1.9.1-dev --no-install-recommends"
    mongonode.vm.provision :shell, :inline => "sudo apt-get install -y ruby1.9.3 --no-install-recommends"
                mongonode.vm.provision :shell, :inline => "sudo gem install cf"
  end  

end
