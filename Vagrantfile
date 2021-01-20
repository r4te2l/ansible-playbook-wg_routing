# -*- mode: ruby -*-
# vi: set ft=ruby :
require 'erb'

Vagrant.require_version ">= 2.0.0"

CONFIG = File.join(File.dirname(__FILE__), ENV['WG_ROUTING'] || 'vagrant/config.rb')

if File.exist?(CONFIG)
  require CONFIG
end

$num_instances ||= 5
$instance_name_prefix ||= "wgr"
$vm_memory ||= 1024
$vm_cpus ||= 2
$subnet ||= "192.0.2"
$box ||= "ubuntu/focal64"
$playbook ||= "WG_Routing.yaml"
$inventory_path ||= "vagrant/inventory/hosts.yaml"
$limit ||= "all"
$verbose ||= "v"
$requirements_file ||= "roles/requirements.yml"

hosts_file ||= <<BLOCK
127.0.0.1	localhost

# The following lines are desirable for IPv6 capable hosts
::1	ip6-localhost	ip6-loopback
fe00::0	ip6-localnet
ff00::0	ip6-mcastprefix
ff02::1	ip6-allnodes
ff02::2	ip6-allrouters
ff02::3	ip6-allhosts
127.0.1.1	ubuntu-focal	ubuntu-focal
BLOCK

for i in 1..$num_instances
  hosts_file = hosts_file + "\n#{$subnet}.#{i+100} #{"%s-%01d" % [$instance_name_prefix, i]}"
end

$script = <<SCRIPT
cat << EOF > /etc/hosts
#{hosts_file}
EOF
SCRIPT

Vagrant.configure("2") do |config|

  config.vm.box = $box

  config.vm.define vm_name = $instance_name_prefix + '-bastion' do |bastion|
    bastion.vm.hostname = vm_name

    bastion.vm.provider :virtualbox do |vb|
      vb.memory = $vm_memory
      vb.cpus = $vm_cpus
      vb.gui = false
      vb.linked_clone = true
      vb.customize ["modifyvm", :id, "--vram", "8"]
      vb.customize ["modifyvm", :id, "--audio", "none"]
    end

    ip = "#{$subnet}.99"
    bastion.vm.network :private_network, ip: ip

    bastion.vm.provision "shell", inline: "swapoff -a"
    bastion.vm.provision "shell", inline: $script
  end

  (1..$num_instances).each do |i|
    config.vm.define vm_name = "%s-%01d" % [$instance_name_prefix, i] do |node|
      node.vm.hostname = vm_name

      node.vm.provider :virtualbox do |vb|
        vb.memory = $vm_memory
        vb.cpus = $vm_cpus
        vb.gui = false
        vb.linked_clone = true
        vb.customize ["modifyvm", :id, "--vram", "8"]
        vb.customize ["modifyvm", :id, "--audio", "none"]
      end

      ip = "#{$subnet}.#{i+100}"
      node.vm.network :private_network, ip: ip

      node.vm.provision "shell", inline: "swapoff -a"
      node.vm.provision "shell", inline: $script

      if i == $num_instances
        node.vm.provision "ansible" do |ansible|
          ansible.playbook = $playbook
          ansible.inventory_path = $inventory_path
          ansible.limit = $limit
          ansible.verbose = $verbose
          ansible.galaxy_role_file = $requirements_file
          ansible.host_key_checking = false
          ansible.raw_arguments = ["--forks=#{$num_instances}", "--flush-cache", "-e ansible_become_pass=vagrant"]
        end
      end
    end
  end
end
