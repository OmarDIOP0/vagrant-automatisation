# Automatisation du Déploiement d'Applications avec Vagrant

Ce projet utilise **Vagrant** pour automatiser le déploiement d'environnements de développement et de production avec Apache, MySQL et d'autres outils nécessaires.

## Prérequis
Avant de commencer, assurez-vous d'avoir installé :
- [Vagrant](https://www.vagrantup.com/downloads)
- [VirtualBox](https://www.virtualbox.org/wiki/Downloads)

## Configuration des Machines Virtuelles
Deux configurations sont disponibles dans ce projet :

### 1. Environnement CentOS (Finance Web App)
- Utilise **CentOS Stream 9**
- Installe **Apache**, **wget**, **vim**, etc.
- Déploie une application web depuis [Tooplate](https://www.tooplate.com/)

#### Démarrer l'environnement :
```sh
vagrant up
```

### 2. Environnement Ubuntu (WordPress)
- Utilise **Ubuntu 20.04 (Focal)**
- Installe **Apache, MySQL, PHP, WordPress**
- Configure WordPress avec une base de données MySQL

#### Démarrer l'environnement :
```sh
vagrant up
```

## Détails des Configurations

### CentOS (Finance Web App)
Fichier `Vagrantfile` :
```ruby
config.vm.box = "eurolinux-vagrant/centos-stream-9"
config.vm.network "private_network", ip: "192.168.56.28"
config.vm.provider "virtualbox" do |vb|
  vb.memory = "1024"
end
config.vm.provision "shell", inline: <<-SHELL
  yum install httpd wget zip unzip vim -y
  systemctl start httpd
  systemctl enable httpd
  mkdir -p /tmp/finance
  cd /tmp/finance
  wget https://www.tooplate.com/zip-templates/2135_mini_finance.zip
  unzip -o 2135_mini_finance.zip
  cp -r 2135_mini_finance/* /var/www/html/
  systemctl restart httpd
  rm -rf /tmp/finance
SHELL
```

### Ubuntu (WordPress)
Fichier `Vagrantfile` :
```ruby
config.vm.box = "ubuntu/focal64"
config.vm.network "private_network", ip: "192.168.56.30"
config.vm.provider "virtualbox" do |vb|
  vb.memory = "1024"
end
config.vm.provision "shell", inline: <<-SHELL
  sudo apt update
  sudo apt install apache2 mysql-server php php-mysql -y
  curl https://wordpress.org/latest.tar.gz | sudo -u www-data tar zx -C /srv/www
  echo "<VirtualHost *:80>\nDocumentRoot /srv/www/wordpress\n</VirtualHost>" | sudo tee /etc/apache2/sites-available/wordpress.conf
  sudo a2ensite wordpress
  sudo systemctl restart apache2
  mysql -u root -e 'CREATE DATABASE wordpress;'
  mysql -u root -e 'CREATE USER wordpress@localhost IDENTIFIED BY "admin123";'
  mysql -u root -e 'GRANT ALL PRIVILEGES ON wordpress.* TO wordpress@localhost;'
  mysql -u root -e 'FLUSH PRIVILEGES;'
SHELL
```

## Arrêt et Destruction des Machines
Pour arrêter une VM :
```sh
vagrant halt
```

Pour détruire une VM :
```sh
vagrant destroy -f
```

## Contribution
Les contributions sont les bienvenues ! Forkez le projet et proposez vos améliorations via des pull requests.

## Licence
Ce projet est sous licence **MIT**.

