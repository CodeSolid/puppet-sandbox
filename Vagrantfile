# -*- mode: ruby -*-
# vi: set ft=ruby :

domain = 'example.com'
box = 'centos/7'
ram = 512

puppet_nodes = [
  {:hostname => 'puppet',  :ip => '172.16.32.10', :box => box, :fwdhost => 8140, :fwdguest => 8140, :ram => ram},
  {:hostname => 'client1', :ip => '172.16.32.11', :box => box},
  {:hostname => 'client2', :ip => '172.16.32.12', :box => box},
]

Vagrant.configure("2") do |config|
  puppet_nodes.each do |node|
    config.vm.define node[:hostname] do |node_config|
      # Need to also install vagrant-vbguest plugin, using:
      # vagrant plugin install vagrant-vbguest
      node_config.vm.synced_folder ".", "/vagrant", type: "virtualbox"
      node_config.vm.box = node[:box]
      node_config.vm.box_url = 'https://atlas.hashicorp.com/' + node_config.vm.box
      node_config.vm.hostname = node[:hostname] + '.' + domain
      node_config.vm.network :private_network, ip: node[:ip]

      if node[:fwdhost]
        node_config.vm.network :forwarded_port, guest: node[:fwdguest], host: node[:fwdhost]
      end

      memory = node[:ram] ? node[:ram] : 256;
      node_config.vm.provider :virtualbox do |vb|
        vb.customize [
          'modifyvm', :id,
          '--name', node[:hostname],
          '--memory', memory.to_s
        ]
      end

      # Added for Centos - for Ubuntu it comes with the box.
      node_config.vm.provision "shell", inline: "yes | sudo rpm -ivh https://yum.puppetlabs.com/puppetlabs-release-el-7.noarch.rpm"
      node_config.vm.provision "shell", inline: "sudo yum -y install puppet"

      node_config.vm.provision :puppet do |puppet|
        puppet.manifests_path = 'provision/manifests'
        puppet.module_path = 'provision/modules'        
      end
    end
  end
end
