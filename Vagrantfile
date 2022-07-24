Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/bionic64"
  config.vm.provider "virtualbox" do |vb|
    vb.cpus = 2
    vb.memory = 1024
  end

  config.vm.define "apache2" do |apache2|
    apache2.vm.provider "virtualbox" do |vb|
      vb.name = "apache2_0"
    end

    apache2.vm.network "public_network", ip: "192.168.2.220"

    apache2.vm.provision  "shell",
      inline: "apt-get update && \
      apt-get install -y software-properties-common && \
      add-apt--repository --yes --update ppa:ondrej/php && \
      apt-get update
      apt-get install -y php apache2 libapache2-mod-php php-gd php-ssh2 php-mysql && \
      rm -rf /var/www/html/index.html && \
      cp -a /vagrant/artifacts/wordpress/. /var/www/html/"
  end

  config.vm.define "mysql" do |mysql|
    mysql.vm.provider "virtualbox" do |vb|
      vb.name = "mysql_0"
    end

    mysql.vm.network "public_network", ip: "192.168.2.221"
  end

  config.vm.define "ansible" do |ansible|
    ansible.vm.provider "virtualbox" do |vb|
      vb.name = "ansible_0"
    end
    
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
      inline: "ansible-playbook -i /home/vagrant/hosts /home/vagrant/provisioning.yml"
  end
end
