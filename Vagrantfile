VAGRANTFILE_API_VERSION = "2"

$install_mysql_cluster1 = <<-SCRIPT
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
   apt install mc -y
   echo -e "192.168.1.11       srv1\n192.168.1.12       srv2\n192.168.1.100       srv\n172.16.94.11       apache-node1\n172.16.94.12       apache-node2\n172.16.94.13       apache-node3\n172.16.94.14       mysql-node1\n172.16.94.15       mysql-node2\n172.16.94.16       mysql-node3">> /etc/hosts
   sed -i 's/127.0.1.1/#127.0.1.1/g' /etc/hosts
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

$install_mysql_cluster = <<-SCRIPT
   nod_nam=$1
   nod_ip=$2
   is_first_node=$3
  echo    ""
  echo    "-----------------------------------------------------------------"
  echo    "|   mariadb section has been selected to install                |"
  echo    "-----------------------------------------------------------------"
  echo    ""   
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
       shell.args = ["mysql-node1","172.16.94.11","true"]
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
       shell.args = ["mysql-node2","172.16.94.12","false"]
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
       shell.args = ["mysql-node3","172.16.94.13","false"]
    end
   end

  config.vm.provider :virtualbox do |vb|
    vb.customize ["modifyvm", :id, "--memory", "320"]
    vb.customize ["modifyvm", :id, "--cpus", "1"]
  end
end
