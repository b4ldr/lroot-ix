# Example 7
#
# Pulling out all the stops with cluster of seven Vagrant boxes.
#
domain   = 'lab.johnbond.org'
nodes = [
  { :hostname => 'lroot-node', :box => 'ubuntu/trusty64', :networks => [
    { :ip => '192.168.0.110', :type => 'public_network' }]},
  { :hostname => 'lroot-host', :box => 'ubuntu/trusty64', :networks => [ 
    { :ip => '172.16.0.111', :type => 'private_network' },
    { :ip => '192.168.0.111', :type => 'public_network' }]},
  { :hostname => 'ix-rs', :box => 'ubuntu/trusty64', :networks => [
    { :ip => '172.16.0.1', :type => 'private_network' }]},
  { :hostname => 'ix-peer', :box => 'ubuntu/trusty64', :networks => [
    { :ip => '172.16.0.2', :type => 'private_network' }]},
]

Vagrant.configure('2') do |config|
  nodes.each_with_index do |node, index|
    config.vm.define node[:hostname] do |node_config|
      node_config.vm.box       = node[:box]
      node_config.vm.host_name = node[:hostname] + '.' + domain
      node[:networks].each do |net|
        node_config.vm.network net[:type], ip: net[:ip]
      end
      if node[:hostname] == 'lroot-node' then
        node_config.vm.provision "shell", 
          inline: "ip addr add 199.7.83.42/24 dev lo || true", preserve_order: true
      end

      node_config.ssh.shell = "bash -c 'BASH_ENV=/etc/profile exec bash'"
      node_config.vm.provision "shell", inline: "apt-get update", preserve_order: true
      node_config.vm.provision "shell", inline: "apt-get install -y puppet git ruby1.9.1-dev", preserve_order: true
      node_config.vm.provision "shell", inline: "gem install librarian-puppet", preserve_order: true
      node_config.vm.provision "shell", inline: "cd /etc/puppet && librarian-puppet clean --verbose", preserve_order: true
      node_config.vm.provision "shell", inline: "rm -f /etc/puppet/Puppetfile*", preserve_order: true
      node_config.vm.provision "shell", inline: "cp /vagrant/puppet/Puppetfile /etc/puppet", preserve_order: true
      node_config.vm.provision "shell", inline: "cd /etc/puppet && librarian-puppet install --verbose", preserve_order: true
      node_config.vm.provision "shell", inline: "echo 1 > /proc/sys/net/ipv4/ip_forward", preserve_order: true
      node_config.vm.provision :puppet do |puppet|
        puppet.manifests_path    = 'puppet/manifests'
        puppet.manifest_file     = 'site.pp'
        puppet.hiera_config_path = 'hiera/hiera.yaml'
      end
    end
  end
end
