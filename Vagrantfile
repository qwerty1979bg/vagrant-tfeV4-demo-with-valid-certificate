# -*- mode: ruby -*-
# vi: set ft=ruby :

# Install vagrant-disksize to allow resizing the vagrant box disk.
unless Vagrant.has_plugin?("vagrant-disksize")
    raise  Vagrant::Errors::VagrantError.new, "vagrant-disksize plugin is missing. Please install it using 'vagrant plugin install vagrant-disksize' and rerun 'vagrant up'"
end

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  config.vm.box = "ubuntu/bionic64"

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

  # Forward the Replicated Dashboard port - TCP 8800
  config.vm.network "forwarded_port", guest: 8800, host: 8800, host_ip: "127.0.0.1"

  # Forwarding the TFE standard HTTPS port - TCP 443 to TCP 8443 on the host
  config.vm.network "forwarded_port", guest: 443, host: 8443, host_ip: "127.0.0.1"

  # Set up the desired root disk size - requires the "vagrant-disksize" plugin
  config.disksize.size = '50GB'

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #

  config.vm.provider "virtualbox" do |vb|
  # Customize the amount of memory on the VM:
    vb.memory = "8192"
  # Customize the virtual CPUs
    vb.cpus = 2
  end

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  config.vm.provision "shell", env: {"TFE_FQDN" => ENV['TFE_FQDN'], "DUCKDNS_TOKEN" => ENV['DUCKDNS_TOKEN']}, inline: <<-SHELL
    echo Debug:
    echo TFE_FQDN: $TFE_FQDN
    echo DUCKDNS_TOKEN: $DUCKDNS_TOKEN

    if [[ $TFE_FQDN == "" ]]
     then
       echo Please provide a value for TFE_FQDN
       exit 1
    fi

    if [[ $DUCKDNS_TOKEN == "" ]]
     then
       echo Please provide a value for DUCKDNS_TOKEN
       exit 1
    fi

  SHELL

  config.vm.provision "shell", env: {"TFE_FQDN" => ENV['TFE_FQDN']}, inline: <<-SHELL
   echo "Checking if the provided FQDN resolves to 127.0.0.1"
   IP_ADDR=$(dig +short $TFE_FQDN)
   if [ $IP_ADDR != "127.0.0.1" ]
    then
      echo "Your FQDN does not point towards 127.0.0.1 but at $IP_ADDR"
      echo "Fix that and try again."
      echo "(You may need to wait a little bit, after making a modification in the duckdns.org webpage)"
      exit 1
   fi
  SHELL

  config.vm.provision "shell", env: {"TFE_FQDN" => ENV['TFE_FQDN'], "DuckDNS_Token" => ENV['DUCKDNS_TOKEN']}, inline: <<-SHELL
   echo "Creating a SSL certificate via Letsencrypt..."
   curl -s -S https://get.acme.sh | sh
   mkdir /cert
   /root/.acme.sh/acme.sh --issue --dns dns_duckdns -d $TFE_FQDN --cert-file /cert/tfe.cer --key-file /cert/tfe.key --ca-file /cert/tfe-ca.cer --fullchain-file /cert/tfe-fullchain.cer
  SHELL

  config.vm.provision "shell", inline: <<-SHELL
   echo "Installing TFE..."
   curl https://install.terraform.io/ptfe/stable | sudo bash
  SHELL
end
