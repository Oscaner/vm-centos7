# -*- mode: ruby -*-
# vi: set ft=ruby :

require_relative 'vagrant'

# Absolute paths on the host machine.
host_project_dir = File.dirname(File.expand_path(__FILE__))
host_config_dir = host_project_dir

# Absolute paths on the guest machine.
guest_project_dir = '/vagrant'
guest_config_dir = guest_project_dir

vconfig = load_config(
  [
    "#{host_config_dir}/config.yml"
  ]
)

ensure_plugins(vconfig['vagrant_plugins'])

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # Set the name of the VM. See: http://stackoverflow.com/a/17864388/100134
  config.vm.define vconfig['vagrant_machine_name']

  # Networking configuration.
  config.vm.hostname = vconfig['vagrant_hostname']
  config.vm.network :private_network,
                    ip: vconfig['vagrant_ip'],
                    auto_network: vconfig['vagrant_ip'] == '0.0.0.0' && Vagrant.has_plugin?('vagrant-auto_network')

  unless vconfig['vagrant_public_ip'].empty?
    config.vm.network :public_network,
                      ip: vconfig['vagrant_public_ip'] != '0.0.0.0' ? vconfig['vagrant_public_ip'] : nil
  end

  # SSH options.
  config.ssh.insert_key = false
  config.ssh.forward_agent = true

  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  config.vm.box = vconfig['vagrant_box']

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Sync the project root directory to /vagrant
  unless vconfig['vagrant_synced_folders'].any? { |synced_folder| synced_folder['destination'] == '/vagrant' }
    vconfig['vagrant_synced_folders'].push(
      'local_path' => host_project_dir,
      'destination' => '/vagrant'
    )
  end

  # Synced folders.
  vconfig['vagrant_synced_folders'].each do |synced_folder|
    options = {
      type: synced_folder.fetch('type', vconfig['vagrant_synced_folder_default_type']),
      rsync__exclude: synced_folder['excluded_paths'],
      rsync__args: ['--verbose', '--archive', '--delete', '-z', '--copy-links', '--chmod=ugo=rwX'],
      id: synced_folder['id'],
      create: synced_folder.fetch('create', false),
      mount_options: synced_folder.fetch('mount_options', []),
      nfs_udp: synced_folder.fetch('nfs_udp', false)
    }
    synced_folder.fetch('options_override', {}).each do |key, value|
      options[key.to_sym] = value
    end
    config.vm.synced_folder synced_folder.fetch('local_path'), synced_folder.fetch('destination'), options
  end

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  # config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
  #   vb.memory = "1024"
  # end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # VMware Fusion.
  config.vm.provider :vmware_fusion do |v, override|
    # HGFS kernel module currently doesn't load correctly for native shares.
    override.vm.synced_folder host_project_dir, '/vagrant', type: 'nfs'

    v.gui = vconfig['vagrant_gui']
    v.vmx['memsize'] = vconfig['vagrant_memory']
    v.vmx['numvcpus'] = vconfig['vagrant_cpus']
  end

  # VirtualBox.
  config.vm.provider :virtualbox do |v|
    v.linked_clone = true
    v.name = vconfig['vagrant_hostname']
    v.memory = vconfig['vagrant_memory']
    v.cpus = vconfig['vagrant_cpus']
    v.customize ['modifyvm', :id, '--natdnshostresolver1', 'on']
    v.customize ['modifyvm', :id, '--ioapic', 'on']
    v.customize ['modifyvm', :id, '--audio', 'none']
    v.gui = vconfig['vagrant_gui']
  end

  # Parallels.
  config.vm.provider :parallels do |p, override|
    override.vm.box = vconfig['vagrant_box']
    p.name = vconfig['vagrant_hostname']
    p.memory = vconfig['vagrant_memory']
    p.cpus = vconfig['vagrant_cpus']
    p.update_guest_tools = true
  end

  # Cache packages and dependencies if vagrant-cachier plugin is present.
  if Vagrant.has_plugin?('vagrant-cachier')
    config.cache.scope = :box
    config.cache.auto_detect = false
    config.cache.enable :apt
    # Cache the composer directory.
    config.cache.enable :generic, cache_dir: '/home/vagrant/.composer/cache'
    config.cache.synced_folder_opts = {
      type: vconfig['vagrant_synced_folder_default_type'],
      nfs_udp: false
    }
  end

  # Enable provisioning with a shell script. Additional provisioners such as
  # Ansible, Chef, Docker, Puppet and Salt are also available. Please see the
  # documentation for more information about their specific syntax and use.
  # config.vm.provision "shell", inline: <<-SHELL
  #   apt-get update
  #   apt-get install -y apache2
  # SHELL
end
