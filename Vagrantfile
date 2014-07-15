# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = '2'

ENV['VAGRANT_DEFAULT_PROVIDER'] = 'virtualbox'
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  # config.omnibus.chef_version = :latest

  config.vm.box = 'vb-ubuntu-1204'

  config.vm.hostname = 'fbnewswirecheck'

  config.vm.network :private_network, ip: '192.168.37.21'

  config.vm.synced_folder '.', '/vagrant', type: 'nfs'

  config.ssh.forward_agent = true

  config.vm.provider :virtualbox do |vb|
    host = RbConfig::CONFIG['host_os']

    # Give VM 1/4 system memory & access to all cpu cores on the host
    if host =~ /darwin/
      cpus = `sysctl -n hw.ncpu`.to_i
      # sysctl returns Bytes and we need to convert to MB
      mem = `sysctl -n hw.memsize`.to_i / 1024 / 1024 / 4
    elsif host =~ /linux/
      cpus = `nproc`.to_i
      # meminfo shows KB and we need to convert to MB
      mem = `grep 'MemTotal' /proc/meminfo | sed -e 's/MemTotal://' -e 's/ kB//'`.to_i / 1024 / 4
    else # sorry Windows folks, I can't help you
      cpus = 2
      mem = 1024
    end

    vb.customize ['modifyvm', :id, '--memory', mem]
    vb.customize ['modifyvm', :id, '--cpus', cpus]
  end

end
