# -*- mode: ruby -*-
# # vi: set ft=ruby :

required_plugins = %w( vagrant-hostmanager vagrant-cachier )
required_plugins.each do |plugin|
  system "vagrant plugin install #{plugin}" unless Vagrant.has_plugin? plugin
end

Vagrant.configure(2) do |config|

  k8nodes=(ENV['K8_NODES'] || 3)
  config.hostmanager.enabled = true
  config.hostmanager.manage_host = false
  config.hostmanager.manage_guest = true
  config.hostmanager.ignore_private_ip = false
  config.hostmanager.include_offline = true

  (1..k8nodes).each do |i|
    config.vm.define "k8s#{i}" do |s|
      s.ssh.forward_agent = true
      #s.vm.box = "ubuntu/xenial64"
      s.vm.box = "bento/ubuntu-16.04"
      s.vm.hostname = "k8s#{i}"
      s.vm.provision :shell, path: "scripts/bootstrap_ansible.sh"
      if i == 1
        s.vm.provision :shell, inline: "PYTHONUNBUFFERED=1 ansible-playbook /vagrant/ansible/k8s-master.yml -c local"
      else
        s.vm.provision :shell, inline: "PYTHONUNBUFFERED=1 ansible-playbook /vagrant/ansible/k8s-worker.yml -c local"
      end
      s.vm.network "private_network", ip: "172.42.42.#{i}", netmask: "255.255.255.0",
        auto_config: true,
        virtualbox__intnet: "k8s-net"
      s.vm.provider "virtualbox" do |v|
        v.linked_clone = true
        v.name = "k8s#{i}"
        v.memory = 8048
        v.gui = false
      end
    end
  end

  if Vagrant.has_plugin?("vagrant-cachier")
    config.cache.scope = :box
		config.cache.synced_folder_opts =
        {
          owner: '_apt',
          group: '_apt'
        }
  end

end
