# -*- mode: ruby -*-
# vi: set ft=ruby :

# Personal credentials for activate repositories as environment variables
RHEL_USER = ENV['RHEL_USER']
RHEL_PASS = ENV['RHEL_PASS']

# Ask for the path of the RHEL ISO
iso_path = ""
if iso_path.strip.empty?
  print "iso_path empty! Please, define absolute path to your RHEL ISO in the Vagrantfile"
end

Vagrant.configure("2") do |config|
  config.vm.box = "generic/rhel9"
  config.vm.synced_folder ".", "/vagrant", type: "virtualbox"

  ### NODE 0 - CONTROL
  config.vm.define "control" do |control|
    control.vm.hostname = "control.ansible.lab"
    control.vm.network "private_network", ip: "192.168.56.10"
   
    #Defining resources
    control.vm.provider "virtualbox" do |vb|
      vb.memory = "2048"
      vb.cpus = 2
      vb.customize ["storageattach", :id, "--storagectl", "IDE Controller", "--port", "1", "--device", "1", "--type", "dvddrive", "--medium", iso_path]
    end
 
    #Automated Ansible instalation
    control.vm.provision "shell", env: {"RHEL_USER" => RHEL_USER, "RHEL_PASS" => RHEL_PASS}, inline: <<-SHELL
      echo "=== 1. Register in Subscription Manager =="
      sudo subscription-manager register --username "$RHEL_USER" --password "$RHEL_PASS" --auto-attach
      echo "=== 2. Activating CodeReady Builder repo ==="
      sudo subscription-manager repos --enable=codeready-builder-for-rhel-9-x86_64-rpms
      echo "=== 3. Installing dependencies and Ansible ==="
      sudo dnf install epel-release -y
      sudo dnf install ansible -y
      echo "=== 4. SSH key ==="
      if [ ! -f /home/vagrant/.ssh/id_rsa ]; then
        ssh-keygen -t rsa -b 4096 -N "" -f /home/vagrant/.ssh/id_rsa
        chown vagrant:vagrant /home/vagrant/.ssh/id_rsa*
      fi
      cp /home/vagrant/.ssh/id_rsa.pub /vagrant/control_key.pub
      echo "=== 5. Generating Dynamic Inventary for Ansible ==="
      cat <<EOF > /home/vagrant/inventory.ini
[control]
localhost ansible_connection=local

[nodes]
EOF
      for i in {1..4}; do
        echo "node\$i ansible_host=192.168.56.1\$i ansible_user=vagrant" >> /home/vagrant/inventory.ini
      done
      chown vagrant:vagrant /home/vagrant/inventory.ini
      echo "=== Control ready ==="
    SHELL
  end

  ### NODES (1 to 4)
  (1..4).each do |i| 
    config.vm.define "node#{i}" do |node|
      node.vm.hostname = "node#{i}.ansible.lab"
      node.vm.network "private_network", ip: "192.168.56.1#{i}"
      node.vm.provider "virtualbox" do |vb|
        vb.memory = "1024"
      node.vm.provision "shell", inline: <<-SHELL
        echo "=== SSH Config ==="
        until [ -f /vagrant/control_key.pub ]; do sleep 2; done # Wait until control creates its key
        cat /vagrant/control_key.pub >> /home/vagrant/.ssh/authorized_keys
        echo "=== Node#{i} ready ==="
      SHELL
      end
    end
  end
  
end
