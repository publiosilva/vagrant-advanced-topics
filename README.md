# Vagrant Advanced Topics

## Create multiple machines

### Set machine default settings

```ruby
Vagrant.configure("2") do |config|
  # Default settings
  config.vm.box = "ubuntu/bionic64"
  config.vm.provider "virtualbox" do |vb|
    vb.cpus = 1
    vb.memory = 512
  end

  config.vm.define "mymachine1" do |mymachine1|
    # This machine has default settings
  end
end
```

### Set specific settings for just one machine

```ruby
Vagrant.configure("2") do |config|
  # Default settings
  config.vm.box = "ubuntu/bionic64"
  config.vm.provider "virtualbox" do |vb|
    vb.cpus = 1
    vb.memory = 512
  end

  config.vm.define "mymachine1" do |mymachine1|
    # This machine has default settings
  end

  config.vm.define "mymachine2" do |mymachine2|
    # This machine has its own settings
    memcached.vm.box = "centos/7"
    config.vm.provider "virtualbox" do |vb|
      vb.cpus = 2
      vb.memory = 1024
      vb.name = "mymachine2"
    end
  end
end
```

## Create synced folder

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/bionic64"

  config.vm.synced_folder "/my_local_folder", "/my_remote_folder"
end
```

### Disable default synced folder

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/bionic64"

  config.vm.synced_folder ".", "/vagrant", disabled: true
end
```

## Forward port

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/bionic64"

  config.vm.network "forwarded_port", guest: 80, host: 8080
end
```

## Set static IP on private network

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/bionic64"

  config.vm.network "private_network", ip: "192.168.2.10"
end
```

## Set static IP on public network

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/bionic64"

  config.vm.network "public_network", ip: "192.168.2.10"
end
```

## Provision with shell

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/bionic64"

  config.vm.define "apache2" do |apache2|
    apache2.vm.network "public_network", ip: "192.168.2.220"

    apache2.vm.provision  "shell",
      inline: "apt-get update && \
      apt-get install -y software-properties-common && \
      add-apt--repository --yes --update ppa:ondrej/php && \
      apt-get update
      apt-get install -y php apache2 libapache2-mod-php php-gd php-ssh2 php-mysql && \
      rm /var/www/html/index.html && \
      cp -a /vagrant/artifacts/wordpress/. /var/www/html/"
  end
end
```

## Provision with ansible

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/bionic64"

  config.vm.define "mysql" do |mysql|
    mysql.vm.network "public_network", ip: "192.168.2.221"
  end

  config.vm.define "ansible" do |ansible|
    ansible.vm.network "public_network", ip: "192.168.2.222"

    ansible.vm.provision  "shell",
      inline: "cp /vagrant/.vagrant/machines/mysql/virtualbox/private_key /home/vagrant/private_key_mysql && \
              chmod 600 /home/vagrant/private_key_mysql && \
              chown vagrant:vagrant /home/vagrant/private_key_mysql"

    ansible.vm.provision  "shell",
      inline: "cp /vagrant/artifacts/hosts /home/vagrant && \
              chmod a-x /home/vagrant/hosts"

    ansible.vm.provision  "shell",
      inline: "cp /vagrant/artifacts/provisioning.yml /home/vagrant"
    
    ansible.vm.provision  "shell",
      inline: "apt-get update && \
              apt-get upgrade && \
              apt-get install -y software-properties-common && \
              add-apt-repository --yes --update ppa:ansible/ansible && \
              apt-get update
              apt install -y ansible"

    ansible.vm.provision "shell",
      inline: "ansible-playbook -i /home/vagrant/hosts \
            /home/vagrant/provisioning.yml"
  end
end
```

Create a file called `hosts` in the same directory with the following content.

```
[database]
192.168.2.221 ansible_user=vagrant ansible_ssh_private_key_file=/home/vagrant/private_key_mysql
```

Create a file called `provisioning.yml` in the same directory with the following content.

```yml
---
- hosts: database
  tasks:
  - name: 'Install packages'
    apt: 
      name: [mysql-server, python3-mysqldb]
      state: latest
    become: yes
  - name: 'Create MySQL database'
    mysql_db:
      name: wordpress_db
      login_user: root
      state: present
    become: yes
  - name: 'Create MySQL user'
    mysql_user:
      login_user: root
      name: wordpress_user
      password: '12345'
      priv: 'wordpress_db.*:ALL'
      state: present
    become: yes
```

## Docker

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/bionic64"

  config.vm.define "docker" do |docker|
    docker.vm.provider "virtualbox" do |vb|
      vb.memory = 512
      vb.cpus = 1
      vb.name = "docker_0"
    end

    docker.vm.provision "shell",
      inline: "apt-get update && apt-get install -y docker.io"
  end
end
```
