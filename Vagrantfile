# -*- mode: ruby -*-
# vi: set ft=ruby :
#===============================================================================
#-- Version     0.1.0-alpha
#-- Revised     20200226-140158
#-- Contact     awmyhr <awmyhr@gmail.com>
#===============================================================================
#-- Project Name        Vagrant - Ansible Tower Cluster
#-- Project Home        https://github.com/awmyhr/vagrant-ansible_tower_cluster
#-- Date Created        2020-02-26
#-- Author(s)           awmyhr <awmyhr@gmail.com>
#-- Template Version    1.0.0
#===============================================================================
#-- NOTE: Some code and lots of inspiration for this file came from:
#       https://github.com/mikecali/libvirt-vagrant-Installing-Tower-Cluster
#       https://fale.io/blog/2017/07/26/ansible-tower-in-vagrant/
#       https://github.com/shanemcd/ansible-tower-vagrant-cluster
#===============================================================================
tower_domain = ENV.fetch('TOWER_DOMAIN', 'atc-demo.net')
tower_nodes  = ENV.fetch('TOWER_NODES', 1).to_i

hardware = {
    :box     => "centos/7",
    :version => "1905.1",
    :ram     => 2096,
    :cpu     => 1
}
ansible_groups = {
    tower: (1..tower_nodes).map { |i| "atc-node-#{i}.#{tower_domain}" },
    database: ["atc-db.#{tower_domain}"],
    haproxy: ["atc.#{tower_domain}"]
}

#-------------------------------------------------------------------------------
#-- Will be created automatically during `vagrant up`
inventory = '.vagrant/provisioners/ansible/inventory/vagrant_ansible_inventory'
inventory_path = File.expand_path(inventory, File.dirname(__FILE__))

#===============================================================================
Vagrant.configure("2") do |config|
    #---------------------------------------------------------------------------
    #-- Configure hardware -  Provider specific configs
    config.vm.box = hardware[:box]
    # config.vm.box_version = hardware[:version]
    config.vm.synced_folder ".", "/vagrant", id: "vagrant-root", disabled: true
    # Provider specific configs
    config.vm.provider :libvirt do |vmp|
        vmp.cpus   = hardware[:cpu]
        vmp.memory = hardware[:ram]
    end
    # config.vm.provider :virtualbox do |vmp|
    #     vmp.memory = hardware[:ram]
    # end
    # config.vm.provider :vmware_fusion do |vmp|
    #     vmp.vmx['memsize'] = hardware[:ram]
    # end

    #---------------------------------------------------------------------------
    #-- Configure proxy
    config.vm.define "atc.#{tower_domain}" do |config|
        config.vm.hostname = "atc.#{tower_domain}"
        # config.vm.network :private_network, ip: "192.168.220.20"
    end

    #---------------------------------------------------------------------------
    #-- Configure nodes
    (1..tower_nodes).each do |node|
        config.vm.define "atc-node-#{node}.#{tower_domain}" do |config|
            config.vm.hostname =  "atc-node-#{node}.#{tower_domain}"
            # config.vm.network :private_network, ip: "192.168.220.#{node + 100}"
        end
    end

    #---------------------------------------------------------------------------
    #-- Configure database
    config.vm.define "atc-db.#{tower_domain}" do |config|
        config.vm.hostname = "atc-db.#{tower_domain}"
        # config.vm.network :private_network, ip: "192.168.220.50"

    #---------------------------------------------------------------------------
    #-- Running ansible provisioning in last machine created to ensure all exist
    #---------------------------------------------------------------------------
        config.vm.provision 'haproxy', type: 'ansible' do |ansible|
            ansible.limit = 'haproxy'
            ansible.groups = ansible_groups
            ansible.playbook = 'ansible/atc-proxy.yaml'
        end

    end
end
