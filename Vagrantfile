IMAGE_NAME = "bento/ubuntu-24.04"

Vagrant.configure("2") do |config|
  config.ssh.insert_key = false

  config.vm.provider "virtualbox" do |zabbix|
    zabbix.memory = 2048
    zabbix.cpus = 2
  end

  config.vm.define "jezj" do |jezj|
    jezj.vm.box = IMAGE_NAME
    jezj.vm.network "forwarded_port", guest: 22, host: 2202
    jezj.vm.network "forwarded_port", guest: 80, host: 8080
    jezj.vm.hostname = "jezj"
  end

  config.vm.provision "shell", inline: <<-SHELL
    # Aktualizace systému
    sudo apt-get update
    sudo apt-get upgrade -y

    # Instalace základních nástrojů
    sudo apt-get install -y wget curl vim gnupg2

    # Instalace MySQL serveru (MariaDB)
    sudo apt-get install -y mariadb-server
    sudo systemctl start mariadb
    sudo systemctl enable mariadb

    # Konfigurace databáze pro Zabbix
    sudo mysql -e "CREATE DATABASE zabbix CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;"
    sudo mysql -e "CREATE USER 'zabbix'@'localhost' IDENTIFIED BY 'zabbix_password';"
    sudo mysql -e "GRANT ALL PRIVILEGES ON zabbix.* TO 'zabbix'@'localhost';"
    sudo mysql -e "SET GLOBAL log_bin_trust_function_creators = 1;"
    sudo mysql -e "FLUSH PRIVILEGES;"

    # Přidání Zabbix 7.0 LTS repository
    wget https://repo.zabbix.com/zabbix/7.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_latest_7.0+ubuntu24.04_all.deb
    sudo dpkg -i zabbix-release_latest_7.0+ubuntu24.04_all.deb
    sudo apt-get update

    # Instalace Zabbix serveru, frontend a agenta
    sudo apt install -y zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent2

    # Import Zabbix databázové struktury
    sudo zcat /usr/share/zabbix-sql-scripts/mysql/server.sql.gz | mysql --default-character-set=utf8mb4 -uzabbix -pzabbix_password zabbix

    # Disable log_bin_trust_function_creators option after importing database schema.
    sudo mysql -e "SET GLOBAL log_bin_trust_function_creators = 0;"

    # Konfigurace Zabbix serveru
    sudo sed -i 's/# DBPassword=/DBPassword=zabbix_password/' /etc/zabbix/zabbix_server.conf

    # Spuštění Zabbix serveru a agenta
    sudo systemctl restart zabbix-server zabbix-agent2 apache2
    sudo systemctl enable zabbix-server zabbix-agent2 apache2

    # Konfigurace PHP pro Zabbix frontend
    sudo sed -i 's/^max_execution_time = .*/max_execution_time = 300/' /etc/php/*/apache2/php.ini
    sudo sed -i 's/^memory_limit = .*/memory_limit = 128M/' /etc/php/*/apache2/php.ini
    sudo sed -i 's/^post_max_size = .*/post_max_size = 16M/' /etc/php/*/apache2/php.ini
    sudo sed -i 's/^upload_max_filesize = .*/upload_max_filesize = 2M/' /etc/php/*/apache2/php.ini
    sudo sed -i 's/^;date.timezone =.*/date.timezone = Europe\\/Prague/' /etc/php/*/apache2/php.ini

    # Restart Apache pro načtení změn
    sudo systemctl restart apache2
  SHELL
end
