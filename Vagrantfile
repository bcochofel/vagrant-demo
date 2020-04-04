# -*- mode: ruby -*-
# vi: set ft=ruby :

# Check for missing plugins
required_plugins = %w(vagrant-hostmanager vagrant-scp vagrant-env)
plugin_installed = false
required_plugins.each do |plugin|
  unless Vagrant.has_plugin?(plugin)
    system "vagrant plugin install #{plugin}"
    plugin_installed = true
  end
end

# If new plugins installed, restart Vagrant process
if plugin_installed === true
  exec "vagrant #{ARGV.join' '}"
end

### configuration parameters ###

# Vagrant variables
VAGRANTFILE_API_VERSION = "2"

# virtual machines to create
servers = [
  {
    :hostname => "server-1",
    :ip => "192.168.77.200",
    :ram => 2048,
    :cpus => 2,
    :box => "bento/ubuntu-18.04"
  }
]

### main ###

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  # vagrant-hostmanager options
  config.hostmanager.enabled = true
  config.hostmanager.manage_host = true
  config.hostmanager.manage_guest = true
  config.hostmanager.ignore_private_ip = false
  config.hostmanager.include_offline = false

  # vagrant-env options
  config.env.enable
  
  # Forward ssh agent to easily ssh into the different machines
  config.ssh.forward_agent = true

  # create servers
  servers.each do |server|
    config.vm.define server[:hostname] do |config|
      # vm definitions
      config.vm.hostname = server[:hostname]
      config.vm.box = server[:box]
      config.vm.network :private_network, ip: server[:ip]

      memory = server[:ram]
      cpus = server[:cpus]

      # providers
      config.vm.provider :virtualbox do |vb|
        vb.customize [
          "modifyvm", :id,
          "--memory", memory.to_s,
          "--cpus", cpus.to_s,
          "--ioapic", "on",
          "--natdnshostresolver1", "on",
          "--natdnsproxy1", "on"
        ]
      end

      # provisioners
      config.vm.provision "step1", type: "shell", run: "never" do |m|
        m.path = "scripts/step1.sh"
      end

      config.vm.provision "step2", type: "shell" do |m|
        m.path = "scripts/step2.sh"
      end

    end
  end
end
