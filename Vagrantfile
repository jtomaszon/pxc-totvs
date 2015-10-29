# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|
  config.vm.box = "centos6.5"
  config.vm.box_check_update = false
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  config.vm.provider "virtualbox" do |vb|
    vb.memory = "512"
  end

  config.vm.define "pxc01" do |pxc01|
    pxc01.vm.hostname = 'pxc01'
    pxc01.vm.network :private_network, ip: '192.168.70.10'
    pxc01.vm.provision "shell", inline: <<-SHELL
      sudo cp /vagrant/my.cnf-pxc01 /etc/my.cnf
      rm -f /var/lock/subsys/mysql
      sudo service mysql bootstrap-pxc
      sudo mysql -e "GRANT RELOAD, LOCK TABLES, REPLICATION CLIENT ON *.* TO 'sst'@'localhost' IDENTIFIED BY 'secret';"
      sudo mysql -e "GRANT USAGE ON *.* TO 'lb'@'192.168.70.100';"
      sudo mysql -e "GRANT PROCESS ON *.* TO 'clustercheckuser'@'localhost' IDENTIFIED BY 'clustercheckpassword!';"
      sudo mysql -e "GRANT ALL ON employees.* TO 'app'@'192.168.70.100' IDENTIFIED BY 'pass';"
      sudo mysql -e "GRANT ALL ON *.* TO 'sysb'@'%' IDENTIFIED BY 'sysb';"
      cd /vagrant/test_db/
      sudo mysql < employees_partitioned.sql
      sudo service xinetd restart
    SHELL
  end
  config.vm.define "pxc02" do |pxc02|
    pxc02.vm.hostname = 'pxc02'
    pxc02.vm.network :private_network, ip: '192.168.70.20'
    pxc02.vm.provision "shell", inline: <<-SHELL
      sudo cp /vagrant/my.cnf-pxc02 /etc/my.cnf
      rm -f /var/lock/subsys/mysql
      sudo service mysql restart
      sudo service xinetd restart
    SHELL
  end
  config.vm.define "pxc03" do |pxc03|
    pxc03.vm.hostname = 'pxc03'
    pxc03.vm.network :private_network, ip: '192.168.70.30'
    pxc03.vm.provision "shell", inline: <<-SHELL
      sudo cp /vagrant/my.cnf-pxc03 /etc/my.cnf
      rm -f /var/lock/subsys/mysql
      sudo service mysql restart
      sudo service xinetd restart
    SHELL
  end

  # Different deploy script for web-server
  config.vm.define "web" do |web|
    web.vm.hostname = 'webserver'
    web.vm.network :private_network, ip: '192.168.70.100'
    web.vm.provision "shell", inline: <<-SHELL
      # configure haproxy for MySQL
      sudo yum install -y haproxy
      sudo cp -af /vagrant/haproxy.cfg /etc/haproxy/haproxy.cfg
      sudo service haproxy restart
      curl --silent --location https://rpm.nodesource.com/setup | sudo bash -
      sudo yum install -y nodejs 
      sudo yum install gcc-c++ make
      # upload your application
      sudo npm install -g npm
      sudo npm install -g pm2
      sudo npm install -g express-generator
      cd /vagrant/express-mysql
      npm install
      pm2 start bin/www --name express
    SHELL
  end

  # Common deploy file for all the VMs
  config.vm.provision "shell", inline: <<-SHELL
    cd /vagrant/rpms
    echo "Installing dependencies"
    sudo rpm -Uvh *.rpm > /dev/null 2>&1
    exit 0
  SHELL

end
