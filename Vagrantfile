VAGRANTFILE_API_VERSION = "2"

$install_test = <<-SCRIPT
  nod_nam=$1
  nod_ip=$2
  is_first_node=$3

  echo $nod_nam
  echo $nod_ip
  echo $is_first_node

  echo "Substitution with sed..."
  sed -i 's/bind-address\t\t= 127.0.0.1/#bind-address\t\t= 127.0.0.1\\nskip-bind-address\\nskip-networking=0\\n/g' /etc/mysql/my.cnf
  echo $?
SCRIPT

$install_misc = <<-SCRIPT
  echo    ""
  echo    "-------------------------------------------------"
  echo    "|   misc section has been selected to install   |"
  echo    "-------------------------------------------------"
  echo    ""

   apt update
   apt install mc sshpass -y
   echo -e "192.168.1.11       srv1\n192.168.1.12       srv2\n192.168.1.100       srv\n172.16.94.14       apache-node1\n172.16.94.15       apache-node2\n172.16.94.16       apache-node3\n172.16.94.11       mysql-node1\n172.16.94.12       mysql-node2\n172.16.94.13       mysql-node3\n172.16.94.17       mysql-slave">> /etc/hosts
   sed -i 's/127.0.1.1/#127.0.1.1/g' /etc/hosts
SCRIPT

$install_apache_php = <<-SCRIPT
  echo    ""
  echo    "------------------------------------------------------------"
  echo    "|   apache2 and php7 section has been selected to install   |"
  echo    "------------------------------------------------------------"
  echo    ""

   add-apt-repository ppa:ondrej/php -y
   apt-get update
   apt-get install php7.2 php7.2-mysql -y
   cp -v /vagrant/apache/wordpress.conf /etc/apache2/sites-available/wordpress.conf
   a2dissite 000-default
   systemctl reload apache2
   a2ensite wordpress
   systemctl reload apache2
   a2enmod rewrite
   systemctl restart apache2
SCRIPT

$install_glusterfs = <<-SCRIPT
  brick_id=$1
  echo    ""
  echo    "------------------------------------------------------------"
  echo    "|   glusterfs section has been selected to install         |"
  echo    "------------------------------------------------------------"
  echo    ""
  echo    "brick id is :"$brick_id

  mkfs.xfs -i size=512 /dev/sdb
  mkdir -pv /data/glusterfs/var_www/brick0${brick_id}
  echo "/dev/sdb /data/glusterfs/var_www/brick0${brick_id} xfs defaults 0 0" >> /etc/fstab
  mount -a
  apt install glusterfs-server -y
  systemctl start glusterd
  systemctl status glusterd

  case $brick_id in
    1)
       gluster peer probe apache-node2
       gluster peer probe apache-node3
       gluster peer status
       gluster volume create var_www replica 3 transport tcp apache-node1:/data/glusterfs/var_www/brick01/brick \
                                                     apache-node2:/data/glusterfs/var_www/brick02/brick \
                                                     apache-node3:/data/glusterfs/var_www/brick03/brick
       gluster volume start var_www
       echo "apache-node${brick_id}:/var_www /var/www glusterfs defaults,_netdev,fetch-attempts=5 0 0" >> /etc/fstab
       mount -a
       sshpass -p 'vagrant' ssh -o StrictHostKeyChecking=no vagrant@apache-node2 'sudo mount -a'
       sshpass -p 'vagrant' ssh -o StrictHostKeyChecking=no vagrant@apache-node3 'sudo mount -a'
       ;;
    2|3)
       echo "apache-node${brick_id}:/var_www /var/www glusterfs defaults,_netdev,fetch-attempts=5 0 0" >> /etc/fstab
       ;;
  esac
SCRIPT

$install_wordpress = <<-SCRIPT
  echo    ""
  echo    "------------------------------------------------------------"
  echo    "|      wordpress section has been selected to install      |"
  echo    "------------------------------------------------------------"
  echo    ""

  mysql -u root -h srv --password=123456 -e "CREATE DATABASE wordpress;"
  mysql -u root -h srv --password=123456 -e "CREATE USER 'wordpress'@'%' IDENTIFIED BY 'wordpress';"
  mysql -u root -h srv --password=123456 -e "GRANT ALL PRIVILEGES ON wordpress.* TO 'wordpress'; FLUSH PRIVILEGES;"

  wget https://wordpress.org/latest.tar.gz -P /tmp/
  tar -xvzf /tmp/latest.tar.gz -C /var/www/
  chown -Rv www-data:www-data /var/www/wordpress
  cp -v /vagrant/apache/wp-config.php /var/www/wordpress/wp-config.php

SCRIPT


$install_keepalived_haproxy = <<-SCRIPT
  node=$1
  echo    ""
  echo    "-----------------------------------------------------------------"
  echo    "|   haproxy section has been selected to install on node: $node|"
  echo    "-----------------------------------------------------------------"
  echo    ""

  sudo apt install keepalived haproxy -y

  rm -fv /etc/haproxy/haproxy.conf
  rm -fv /etc/keepalived/keepalived.conf

  cp -pv /vagrant/srv${node}/haproxy.cfg /etc/haproxy/
  cp -pv /vagrant/srv${node}/keepalived.conf /etc/keepalived/
  chmod a-x /etc/keepalived/keepalived.conf

  #allow binding to virtual IP of keepalived and apply this adjustment
  echo "net.ipv4.ip_nonlocal_bind=1" >> /etc/sysctl.conf
  sysctl -p

  systemctl daemon-reload
  systemctl start keepalived
  systemctl enable keepalived
  systemctl restart haproxy
  systemctl enable haproxy
SCRIPT

$install_mysql_slave = <<-SCRIPT
   nod_nam=$1
   nod_ip=$2
   is_first_node=$3

   echo    ""
   echo    "-----------------------------------------------------------------"
   echo    "|   mariadb slave section has been selected to install          |"
   echo    "-----------------------------------------------------------------"
   echo    ""

   sudo apt-key adv --recv-keys --keyserver hkp://keyserver.ubuntu.com:80 0xF1656F24C74CD1D8
   sudo add-apt-repository 'deb [arch=amd64] http://nyc2.mirrors.digitalocean.com/mariadb/repo/10.4/ubuntu bionic main'
   sudo apt update
   sudo apt install mariadb-server rsync mc -y
   mysql -uroot -e 'set password = password("123456");'
SCRIPT

$install_mysql_cluster = <<-SCRIPT
   nod_nam=$1
   nod_ip=$2
   is_first_node=$3
   gtid=$4
   echo    ""
   echo    "-----------------------------------------------------------------"
   echo    "|   mariadb section has been selected to install                |"
   echo    "-----------------------------------------------------------------"
   echo    ""

   sudo apt-key adv --recv-keys --keyserver hkp://keyserver.ubuntu.com:80 0xF1656F24C74CD1D8
   sudo add-apt-repository 'deb [arch=amd64] http://nyc2.mirrors.digitalocean.com/mariadb/repo/10.4/ubuntu bionic main'
   sudo apt update
   sudo apt install mariadb-server rsync mc -y
   mysql -uroot -e 'set password = password("123456");'

cat > /etc/mysql/conf.d/galera.cnf << "END"
[mysqld]
binlog_format=ROW
default-storage-engine=innodb
innodb_autoinc_lock_mode=2
bind-address=0.0.0.0

# Galera Provider Configuration
wsrep_on=ON
wsrep_provider=/usr/lib/galera/libgalera_smm.so

# Galera Cluster Configuration
wsrep_cluster_name="test_cluster"
wsrep_cluster_address="gcomm://172.16.94.11,172.16.94.12,172.16.94.13"

# Galera Synchronization Configuration
wsrep_sst_method=rsync

# Galera Node Configuration
wsrep_node_address="node_ip"
wsrep_node_name="nm"

#master-slave replication
# Galera node as master
wsrep_gtid_mode      = on
wsrep_gtid_domain_id = 0
server-id            = 01
log_slave_updates    = on
log-bin              = /var/log/mysql/master-bin
log-bin-index        = /var/log/mysql/master-bin.index
gtid_domain_id       = "GTID"
END
#sed -i 's/node_name/'$(hostname)'/g' /etc/mysql/conf.d/galera.cnf
#sed -i 's/node_ip/'$(hostname -I|awk '''{print $3}''')'/g' /etc/mysql/conf.d/galera.cnf
  echo $nod_nam
  echo $nod_ip
  echo $is_first_node

  sed -i 's/nm/'$nod_nam'/g' /etc/mysql/conf.d/galera.cnf
  sed -i 's/node_ip/'$nod_ip'/g' /etc/mysql/conf.d/galera.cnf
  sed -i 's/GTID/'$gtid'/g' /etc/mysql/conf.d/galera.cnf

  sudo systemctl stop mysql

  if [ $is_first_node == true ];then
    #setup new cluster
    sudo galera_new_cluster
    #set cron job that unblocks connectons to cluster that is a matter of extensive haproxy checks
    (crontab -l 2>/dev/null; echo "* * * * * /usr/bin/mysqladmin flush-hosts") | crontab -
  fi

  #allow MariaDb to listen on network interface instead of localhost
  sed -i 's/bind-address\t\t= 127.0.0.1/#bind-address\t\t= 127.0.0.1\\nskip-bind-address\\nskip-networking=0\\n/g' /etc/mysql/my.cnf

  sudo systemctl start mysql

  #this is a cluster, so we need to run these commands only once
  if [ $is_first_node == true ];then
    #enable connections from the network with haproxy instances
    mysql -u root --password=123456 -e "GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;FLUSH PRIVILEGES;"

    #add account for haproxy monitoring
    mysql -u root --password=123456 -e "CREATE USER 'haproxy_user'@'%';GRANT ALL PRIVILEGES ON *.* TO 'haproxy_user'@'%';FLUSH PRIVILEGES;"
  fi

  mysql -u root --password=123456 -e "SHOW STATUS LIKE 'wsrep_cluster_size'"
SCRIPT


Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  config.vm.define "srv1" do |srv1|
    srv1.vm.box = "bento/ubuntu-18.04"
    srv1.vm.hostname = "srv1"
    srv1.vm.network :private_network, ip: "192.168.1.11"
    config.vm.provision "misc", type: "shell" do |shell|
       shell.inline = $install_misc
    end
    config.vm.provision "haproxy", type: "shell" do |shell|
       shell.inline = $install_keepalived_haproxy
       shell.args = ["1"]
    end
  end

  config.vm.define "srv2" do |srv2|
    srv2.vm.box = "bento/ubuntu-18.04"
    srv2.vm.hostname = "srv2"
    srv2.vm.network :private_network, ip: "192.168.1.12"
    config.vm.provision "misc", type: "shell" do |shell|
       shell.inline = $install_misc
    end
    config.vm.provision "haproxy", type: "shell" do |shell|
       shell.inline = $install_keepalived_haproxy
       shell.args = ["2"]
    end
  end

  config.vm.define "mysql1" do |mysql1|
    mysql1.vm.box = "bento/ubuntu-18.04"
    mysql1.vm.hostname = "mysql-node1"
    mysql1.vm.network :private_network, ip: "172.16.94.11"
    config.vm.provision "misc", type: "shell" do |shell|
       shell.inline = $install_misc
    end
       config.vm.provision "mysql_cluster", type: "shell" do |shell|
       shell.inline = $install_mysql_cluster
       shell.args = ["mysql-node1","172.16.94.11","true","1"]
    end
  end

  config.vm.define "mysql2" do |mysql2|
    mysql2.vm.box = "bento/ubuntu-18.04"
    mysql2.vm.hostname = "mysql-node2"
    mysql2.vm.network :private_network, ip: "172.16.94.12"
    config.vm.provision "misc", type: "shell" do |shell|
       shell.inline = $install_misc
    end
       config.vm.provision "mysql_cluster", type: "shell" do |shell|
       shell.inline = $install_mysql_cluster
       shell.args = ["mysql-node2","172.16.94.12","false","2"]
    end
  end

  config.vm.define "mysql3" do |mysql3|
    mysql3.vm.box = "bento/ubuntu-18.04"
    mysql3.vm.hostname = "mysql-node3"
    mysql3.vm.network :private_network, ip: "172.16.94.13"
    config.vm.provision "misc", type: "shell" do |shell|
       shell.inline = $install_misc
    end
    config.vm.provision "mysql_cluster", type: "shell" do |shell|
       shell.inline = $install_mysql_cluster
       shell.args = ["mysql-node3","172.16.94.13","false","3"]
    end
   end

  config.vm.define "mysql4" do |mysql4|
    mysql4.vm.box = "bento/ubuntu-18.04"
    mysql4.vm.hostname = "mysql-slave"
    mysql4.vm.network :private_network, ip: "172.16.94.17"
    config.vm.provision "misc", type: "shell" do |shell|
       shell.inline = $install_misc
    end
    config.vm.provision "mysql_cluster", type: "shell" do |shell|
       shell.inline = $install_mysql_slave
    end
   end


  config.vm.define "apache1" do |apache1|
    apache1.vm.box = "bento/ubuntu-18.04"
    apache1.vm.hostname = "apache-node1"
    apache1.vm.network :private_network, ip: "172.16.94.14"

    config.vm.provider "virtualbox" do |vb|
      second_disk = "apache1_second_disk.vmdk"
      unless File.exist?(second_disk)
        vb.customize ['createhd', '--filename', second_disk, '--variant', 'Fixed', '--size', 1 * 1024]
      end
      vb.customize ['storageattach', :id,  '--storagectl', 'SATA Controller', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', second_disk]
    end

    config.vm.provision "misc", type: "shell" do |shell|
       shell.inline = $install_misc
    end

    config.vm.provision "apache_php", type: "shell", keep_color: "true" do |shell|
       shell.inline = $install_apache_php
    end

    config.vm.provision "glusterfs", type: "shell" do |shell|
       shell.inline = $install_glusterfs
       shell.args = ["1"]
    end

    config.vm.provision "wordpress", type: "shell", keep_color: "true" do |shell|
       shell.inline = $install_wordpress
    end
   end

  config.vm.define "apache2" do |apache2|
    apache2.vm.box = "bento/ubuntu-18.04"
    apache2.vm.hostname = "apache-node2"
    apache2.vm.network :private_network, ip: "172.16.94.15"

    config.vm.provider "virtualbox" do |vb|
      second_disk = "apache2_second_disk.vmdk"
      unless File.exist?(second_disk)
        vb.customize ['createhd', '--filename', second_disk, '--variant', 'Fixed', '--size', 1 * 1024]
      end
      vb.customize ['storageattach', :id,  '--storagectl', 'SATA Controller', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', second_disk]
    end


    config.vm.provision "misc", type: "shell" do |shell|
       shell.inline = $install_misc
    end

    config.vm.provision "apache_php", type: "shell" do |shell|
       shell.inline = $install_apache_php
    end

    config.vm.provision "glusterfs", type: "shell" do |shell|
       shell.inline = $install_glusterfs
       shell.args = ["2"]
    end
   end

  config.vm.define "apache3" do |apache3|
    apache3.vm.box = "bento/ubuntu-18.04"
    apache3.vm.hostname = "apache-node3"
    apache3.vm.network :private_network, ip: "172.16.94.16"

    config.vm.provider "virtualbox" do |vb|
      second_disk = "apache3_second_disk.vmdk"
      unless File.exist?(second_disk)
        vb.customize ['createhd', '--filename', second_disk, '--variant', 'Fixed', '--size', 1 * 1024]
      end
      vb.customize ['storageattach', :id,  '--storagectl', 'SATA Controller', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', second_disk]
    end


    config.vm.provision "misc", type: "shell" do |shell|
       shell.inline = $install_misc
    end

    config.vm.provision "apache_php", type: "shell" do |shell|
       shell.inline = $install_apache_php
    end

    config.vm.provision "glusterfs", type: "shell" do |shell|
       shell.inline = $install_glusterfs
       shell.args = ["3"]
    end
   end

  config.vm.provider :virtualbox do |vb|
    vb.customize ["modifyvm", :id, "--memory", "320"]
    vb.customize ["modifyvm", :id, "--cpus", "1"]
  end
end
