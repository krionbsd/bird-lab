# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  # Use Oracle Linux 7.3 as base image
  config.vm.box = "centos/7"

  # Fail if vagrant-vbguest if not installed.
  # vagrant plugin install vagrant-vbguest
  unless Vagrant.has_plugin?('vagrant-vbguest')
    raise 'vagrant-vbguest not installed!'
  end

  # Customize the hostsmanager ip_resolver
  # to retrive the proper ip address from the
  # guest.
  if Vagrant.has_plugin?("vagrant-hostmanager")
    config.hostmanager.ip_resolver = proc do |machine|
      result = ""
      machine.communicate.execute("ip addr | grep 'dynamic' | tail -1") do |type, data|
        result << data if type == :stdout
      end
      (ip = /inet (\d+\.\d+\.\d+\.\d+)/.match(result)) && ip[1]
    end
  end

  # Use the vbguests coming with the box image
  config.vbguest.auto_update = false 

  # Disable automatic box update checking.
  # Check with `vagrant box outdated` whether the currently
  # installed centos/7 box is outdated.
  config.vm.box_check_update = false

  # Do not forward any port from the guest to the host
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Each machine will have two nics attached to the private
  # network and configured via DHCP
  config.vm.network "private_network", type: "dhcp"

  # Do not share the /vagrant folder
  config.vm.synced_folder ".", "/vagrant", disabled: true

  # Virtualbox specific settings.
  config.vm.provider "virtualbox" do |vb|
    # Customize the amount of memory on the VM
    vb.memory = "1024"
  end

  # if vagrant-hostsupdater is installed,
  # automatically update the /etc/hosts file
  # vagrant plugin install vagrant-hostmanager
  if Vagrant.has_plugin?("vagrant-hostmanager")
    config.hostmanager.enabled = true
    config.hostmanager.manage_host = true
    config.hostmanager.manage_guest = true
  end

  VMS = [ "bird-serv" ]

  groups = {
    "vagrant" => [ "bird-serv" ]
  }

  VMS.each do |vm|
    config.vm.define "%s" % vm do |bird|

      bird.vm.hostname = vm

      if Vagrant.has_plugin?("vagrant-hostmanager")
        bird.hostmanager.aliases = [ "%s.dev" % vm.delete("bird-serv") ]
      end

      if vm == VMS.last
        # Start Ansible Provisioning
        bird.vm.provision :ansible do |ansible|
          ansible.playbook = "playbook.yml"
          ansible.groups = groups
          ansible.limit = "all"
          ansible.raw_arguments = ['-v', '--diff']
          ansible.raw_ssh_args = ['-o ControlMaster=no']
        end
      end

    end
  end

end
