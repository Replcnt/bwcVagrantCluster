# -*- mode: ruby -*-
# vi: set ft=ruby :
boxes = [
	{ :name => :nimbus, :ip => '192.168.222.100', :cpus => 2, :memory => 1024 },
	{ :name => :supervisor1, :ip => '192.168.222.101', :cpus => 4, :memory => 2048 },
	{ :name => :supervisor2, :ip => '192.168.222.102', :cpus => 4, :memory => 2048 },
	{ :name => :zookeeper1, :ip => '192.168.222.201', :cpus => 1, :memory => 1024 }
]

Vagrant.configure("2") do |config|

boxes.each do |opts|
	config.vm.define opts[:name] do |config|
		config.vm.host_name = "bwc.%s" % opts[:name].to_s
		config.vm.synced_folder "./data", "/vagrant_data"

		config.vm.provider :virtualbox do |vb|
			config.vm.box = "ubuntu12"
			config.vm.box.url = "http://127.0.0.1/precise64.box"
			config.vm.network :private_network, ip: opts[:ip]
			vb.name = "bwc.%s" % opts[:name].to_s
			vb.customize ["modifyvm", :id, "--memory", opts[:memory]]
  		vb.customize ["modifyvm", :id, "--cpus", opts[:cpus] ] if opts[:cpus]
		end
	
		config.vm.provision :shell, :inline => "hostname bwc.%s" % opts[:name].to_s
		config.vm.provision :shell, :inline => "cp -fv /vagrant_data/hosts /etc/hosts"
		config.vm.provision :shell, :inline => "apt-get update"
		config.vm.provision :shell, :inline => "apt-get --yes --force-yes install puppet"

		# Check if the JDK has been provided
		if File.exist?("./data/jdk-6u35-linux-x64.bin") then
			config.vm.provision :puppet do |puppet|
				puppet.manifests_path = "manifests"
				puppet.manifest_file = "jdk.pp"
			end
		end

		config.vm.provision :puppet do |puppet|
			puppet.manifests_path = "manifests"
			puppet.manifest_file = "provisioningInit.pp"
		end

		# Ask puppet to provision VM
		config.vm.provision :shell, :inline => "puppet apply /tmp/storm-puppet/manifests/site.pp --verbose --modulepath=/tmp/storm-puppet/modules/ --debug"
	end
end

end

