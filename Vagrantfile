# -*- mode: ruby -*-
# vi:set ft=ruby sw=2 ts=2 sts=2:
NUM_NODE = 5
KUBEADM_POD_NETWORK_CIDR = "10.244.0.0/16"
KUBEADM_TOKEN = "29c663.4e1b73743dfdcaf1"
KUBEADM_TOKEN_TTL = 0

MASTER_IP = "192.168.26.10"
NODE_IP_NW = "192.168.26."

def get_environment_variable(envvar)
  ENV[envvar] || ENV[envvar.upcase] || ENV[envvar.downcase]
end

def apply_vm_hardware_customizations(provider)
  provider.linked_clone = true

  provider.customize ["modifyvm", :id, "--vram", "16"]
  provider.customize ["modifyvm", :id, "--largepages", "on"]
  provider.customize ["modifyvm", :id, "--nestedpaging", "on"]
  provider.customize ["modifyvm", :id, "--vtxvpid", "on"]
  provider.customize ["modifyvm", :id, "--hwvirtex", "on"]
  provider.customize ["modifyvm", :id, "--ioapic", "on"]
  provider.customize ["modifyvm", :id, "--uart1", "0x3F8", "4"]
  provider.customize ["modifyvm", :id, "--uart2", "0x2F8", "3"]
  provider.customize ["modifyvm", :id, "--uartmode1", "disconnected"]
  provider.customize ["modifyvm", :id, "--uartmode2", "disconnected"]
end

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  http_proxy = get_environment_variable('http_proxy')
  https_proxy = get_environment_variable('https_proxy')
  no_proxy = get_environment_variable('no_proxy')

  if Vagrant.has_plugin?("vagrant-proxyconf") && (http_proxy || https_proxy || no_proxy)
    # copy proxy settings from the Vagrant environment into the VMs
    (no_proxy = [no_proxy, master_vm_ip].join(',')) unless no_proxy.nil?

    # vagrant-proxyconf will not configure anything if everything is nil
    config.proxy.http = http_proxy
    config.proxy.https = https_proxy
    config.proxy.no_proxy = no_proxy

    # look for another envvar that points to a directory with CA certs
    env_cacerts_dir = get_environment_variable('cacerts_dir')
    if env_cacerts_dir
      cacerts_dir = Dir.new(env_cacerts_dir)
      files_in_cacerts_dir = cacerts_dir.entries.select{ |e| not ['.', '..'].include? e }
      files_in_cacerts_dir.each do |f|
        next if File.directory?(File.join(cacerts_dir, f))
        begin
          unless f.end_with? '.crt'
            fail "All files in #{env_cacerts_dir} must end in .crt due to update-ca-certificates restrictions."
          end
          # read in the certificate and normalize DOS line endings to UNIX
          cert_raw = File.read(File.join(env_cacerts_dir, f)).gsub(/\r\n/, "\n")
          if cert_raw.scan('-----BEGIN CERTIFICATE-----').length > 1
            fail "Multiple certificates detected in #{File.join(env_cacerts_dir, f)}, please split them into separate certificates."
          end
          cert = OpenSSL::X509::Certificate.new(cert_raw) # test that the cert is valid
          dest_cert_path = File.join('/usr/local/share/ca-certificates', f)
          update_ca_certificates_script << <<-EOH
            echo -ne "#{cert_raw}" > #{dest_cert_path}
          EOH
        rescue OpenSSL::X509::CertificateError
          fail "Certificate #{File.join(env_cacerts_dir, f)} is not a valid PEM certificate, aborting."
        end # begin/rescue
      end # files_in_cacerts_dir.each
      update_ca_certificates_script << <<-EOH
        update-ca-certificates
      EOH
    end # if env_cacerts_dir
  end # if Vagrant.has_plugin?

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  # config.vm.box = "base"
  config.vm.box = "ubuntu/xenial64"

  # The following config requires hostmanager:
  # config.hostmanager.enabled = true
  # config.hostmanager.manage_guest = true

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  config.vm.box_check_update = false

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

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  config.vm.provider "virtualbox" do |vb|
    # Customize the amount of memory on the VM:
    vb.memory = "2048"
  end
  #
  # View the documentation for the provider you are using for more
  # information on available options.
  config.vm.provision "install-docker", type: "shell", :path => "ubuntu/install-docker.sh"
  config.vm.provision "install-kubeadm", type: "shell", :path => "ubuntu/install-kubeadm.sh"
  config.vm.provision "setup-hosts", :type => "shell", :path => "ubuntu/vagrant/setup-hosts.sh" do |s|
    s.args = ["enp0s8"]
  end

  # config.vm.define "base" do |base|
  #   base.vm.hostname = "master"
  #   base.vm.provision "install-kubeconfig", :type => "shell", :path => "ubuntu/vagrant/install-guest-additions.sh"
  # end

  config.vm.define "master" do |node|
    node.vm.hostname = "master"
    node.vm.network :private_network, ip: MASTER_IP

    # cpu_count = 1
    # env_cpu_count = ENV['master_cpu_count'].to_i
    # (cpu_count = env_cpu_count) unless env_cpu_count.zero?

    # memory_mb = 1024
    # env_memory_mb = ENV['master_memory_mb'].to_i
    # (memory_mb = env_memory_mb) unless env_memory_mb.zero?

    # node.vm.provider "virtualbox" do |vb|
    #   vb.cpus = cpu_count
    #   vb.memory = memory_mb

    #   apply_vm_hardware_customizations(vb)
    # end

    node.vm.provision "setup-hosts", :type => "shell", :path => "ubuntu/vagrant/setup-hosts.sh" do |s|
      s.args = ["enp0s8"]
    end

    node.vm.provision "shell", inline: <<-SHELL
        kubeadm init \
            --apiserver-advertise-address=#{MASTER_IP} \
            --pod-network-cidr=#{KUBEADM_POD_NETWORK_CIDR} \
            --token #{KUBEADM_TOKEN} --token-ttl #{KUBEADM_TOKEN_TTL}
    SHELL

    # use sudo here so that the id subshells get the non-root user
    node.vm.provision 'copy-kubeadm-config', type: 'shell' do |s|
      s.privileged = false
      s.inline = <<-EOH
        #!/bin/sh
        mkdir -p $HOME/.kube
        sudo cp /etc/kubernetes/admin.conf $HOME/.kube/config
        sudo cp -f /etc/kubernetes/admin.conf /vagrant/kube.config
        sudo chown $(id -u):$(id -g) $HOME/.kube/config
      EOH
    end

    node.vm.provision "install-kubeconfig", :type => "shell", :path => "ubuntu/vagrant/install-kubeconfig.sh"
    node.vm.provision "allow-bridge-nf-traffic", :type => "shell", :path => "ubuntu/allow-bridge-nf-traffic.sh"
    node.vm.provision "install-flannel", :type => "shell", :path => "ubuntu/vagrant/install-flannel.sh"
  end

  (1..NUM_NODE).each do |i|

    # cpu_count = 1
    # env_cpu_count = ENV['worker_cpu_count'].to_i
    # (cpu_count = env_cpu_count) unless env_cpu_count.zero?

    # memory_mb = 1024
    # env_memory_mb = ENV['worker_memory_mb'].to_i
    # (memory_mb = env_memory_mb) unless env_memory_mb.zero?

    config.vm.define "node-#{i}" do |node|
        node.vm.hostname = "node-#{i}"
        node.vm.network :private_network, ip: NODE_IP_NW + "#{10 + i}"

        # node.vm.provider "virtualbox" do |vb|
        #   vb.cpus = cpu_count
        #   vb.memory = memory_mb

        #   apply_vm_hardware_customizations(vb)
        # end

        node.vm.provision "allow-bridge-nf-traffic", :type => "shell", :path => "ubuntu/allow-bridge-nf-traffic.sh"

        # --discovery-token-unsafe-skip-ca-verification might be required for custom generated token
        node.vm.provision "shell", inline: <<-SHELL
        kubeadm join --token #{KUBEADM_TOKEN} #{MASTER_IP}:6443 --discovery-token-unsafe-skip-ca-verification
        SHELL
    end
  end
end
