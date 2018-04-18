
# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'yaml'

#for debian8 box
#vagrant plugin install vagrant-vbguest
#vagrant plugin install vagrant-hostsupdater

Vagrant.configure(2) do |config|
  config.vm.box = "debian/jessie64"
  config.vm.box_check_update = false

  config.vm.network "private_network", ip: "192.168.50.95"
  config.vm.hostname = "php7.app"

  config.vm.synced_folder "./", "/home/project", mount_options: ["dmode=777","fmode=777"]

  config.vm.provider "virtualbox" do |vb|
    vb.cpus = 3
    vb.memory = 2024
  end

  config.vm.provision "shell", inline: <<-SHELL
    apt-get update && apt-get upgrade -y

    wget https://repo.percona.com/apt/percona-release_0.1-4.$(lsb_release -sc)_all.deb
    dpkg -i percona-release_0.1-4.$(lsb_release -sc)_all.deb
    apt-get update
    debconf-set-selections <<< 'percona-server-server-5.6 percona-server-server/root_password password dbpass'
    debconf-set-selections <<< 'percona-server-server-5.6 percona-server-server/root_password_again password dbpass'
    apt-get install -y percona-server-server-5.6
    sed -i -e 's/^bind-address.*=.*$/bind-address = 0.0.0.0/' /etc/mysql/my.cnf
    mysql -uroot -pdbpass -e "GRANT ALL PRIVILEGES ON *.* TO 'root'@'%';FLUSH PRIVILEGES;"
    service mysql restart

    apt-get install apt-transport-https lsb-release ca-certificates
    wget -O /etc/apt/trusted.gpg.d/php.gpg https://packages.sury.org/php/apt.gpg
    echo "deb https://packages.sury.org/php/ $(lsb_release -sc) main" > /etc/apt/sources.list.d/php.list
    apt-get update
    apt-get install -y php7.1 php7.1-mysql php7.1-xml php7.1-intl
    sed -i -e 's/^;date.timezone =$/date.timezone = "Europe\/Moscow"/' /etc/php/7.1/apache2/php.ini
    sed -i -e 's/^;date.timezone =$/date.timezone = "Europe\/Moscow"/' /etc/php/7.1/cli/php.ini

    a2dissite 000-default.conf
    echo '<VirtualHost *:80>' > /etc/apache2/sites-available/vhost.conf
    echo '   ServerName php7.app' >> /etc/apache2/sites-available/vhost.conf
    echo '   DocumentRoot /home/project/web' >> /etc/apache2/sites-available/vhost.conf
    echo '   <Directory /home>' >> /etc/apache2/sites-available/vhost.conf
    echo '       Options FollowSymLinks' >> /etc/apache2/sites-available/vhost.conf
    echo '       AllowOverride All' >> /etc/apache2/sites-available/vhost.conf
    echo '       Require all granted' >> /etc/apache2/sites-available/vhost.conf
    echo '   </Directory>' >> /etc/apache2/sites-available/vhost.conf
    echo '</VirtualHost>' >> /etc/apache2/sites-available/vhost.conf
    a2ensite vhost.conf
    a2enmod rewrite
    service apache2 restart
 SHELL
end
