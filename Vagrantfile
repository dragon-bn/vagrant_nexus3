# File              : Vagrantfile
# Author            : Your Name <your@mail>
# Date              : 20.06.2019
# Last Modified Date: 20.06.2019
# Last Modified By  : Your Name <your@mail>
# -*- mode: ruby -*-
# vi: set ft=ruby :

#https://scotch.io/tutorials/how-to-create-a-vagrant-base-box-from-an-existing-one
#https://linoxide.com/linux-how-to/setup-centos-7-vagrant-base-box-virtualbox/

# Specify Vagrant version, Vagrant API version, and desired clone location
Vagrant.require_version '>= 2.2.0'
VAGRANTFILE_API_VERSION = '2'
ENV['VAGRANT_DEFAULT_PROVIDER'] = 'virtualbox'
ENV['VAGRANT_VIRTUALBOX_CLONE_DIRECTORY'] = '~/.vagrant'

# Require 'yaml' module
require 'yaml'
# require_relative 'lib/drupalvm/vagrant'


###############################
# CUSTOM CONFIGURATION START
###############################

# here is where you download your software, so it will be available to the VMs.
#sw_path  = "C:\\Users\\ludov\\Downloads\\Software"
sw_path = "./Data"

# lab_name is the name of the lab where all the files will be organized.
lab_name = "My_Lab"

# Read YAML file with VM details (box, CPU, and RAM)
machines = YAML.load_file(File.join(File.dirname(__FILE__), 'inventories/machines.yml'))

###############################
# CUSTOM CONFIGURATION END
###############################

########
# MAIN #
########

# Create and configure the VMs
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  # Always use Vagrant's default insecure key
  #config.ssh.insert_key = false
  config.ssh.insert_key = true

  config.ssh.insert_key = false
  config.vm.boot_timeout = 800

  config.vm.graceful_halt_timeout = 360	# in case you install grid infra... do not force shutdown after a few seconds

#  if File.directory?(sw_path)
    # our shared folder for oracle 12c installation files (uid 54320 is grid, uid 54321 is oracle)
#    config.vm.synced_folder sw_path, "/media/sw", :mount_options => ["dmode=775","fmode=775","uid=54322","gid=54328"]
#  end

  #config.vm.network :public_network, bridge: "wlp6s0"

  # Iterate through entries in YAML file to create VMs
  machines.each do |machine|
    config.vm.define machine['name'] do |srv|

      # Don't check for box updates
      srv.vm.box_check_update = false

      srv.vm.hostname = machine['name']
      srv.vm.box = machine['box']

      # Configure default synced folder (disable by default)
      if machine['sync_disabled'] != nil
        srv.vm.synced_folder '.', '/vagrant', disabled: machine['sync_disabled']
      else
        srv.vm.synced_folder '.', '/vagrant', disabled: true
      end #if machine['sync_disabled']

      # Assign additional private network
      if machine['ip_addr'] != nil
        srv.vm.network 'private_network', ip: machine['ip_addr']
      end # if machine['ip_addr']

      # Configure CPU & RAM per settings in machines.yml
      srv.vm.provider :virtualbox do |vb|
        vb.name = machine['name']

        # Set VM memory size
        vb.customize ["modifyvm", :id, "--memory", machine['ram']]

        # Set VM cpu size
        vb.customize ["modifyvm", :id, "--cpus", machine['vcpu']]

        # By default, VirtualBox machines are started in headless mode,
        # meaning there is no UI for the machines visible on the host machine.
        vb.gui = false

        # these 2 commands massively speed up DNS resolution, which means outbound
        # connections don't take forever (eg the WP admin dashboard and update page)
        #vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
        #vb.customize ["modifyvm", :id, "--natsproxy1", "on"]

        # Set Group for virtual machin
        vb.customize ["modifyvm", :id, "--groups", "/AutoFormation/MyLab"]

        # Get disk path
        line = `VBoxManage list systemproperties | grep "Default machine folder"`
        vb_machine_folder = line.split(':')[1].strip()
        second_disk = File.join(vb_machine_folder, vb.name, 'disk2.vdi')

        # Attach another hard disk
        # Note: Create a hard disk image: vboxmanage createmedium --filename $PWD/disk-2.vdi --size 1024 --format VDI

        # Create and attach disk
        unless File.exist?(second_disk)
          vb.customize ['createhd', '--filename', second_disk, '--format', 'VDI', '--size', 60 * 1024]
        #end

        # see https://www.virtualbox.org/manual/ch08.html#vboxmanage-storageattach
        vb.customize [ 'storageattach',
           :id, # the id will be replaced (by vagrant) by the identifier of the actual machine
           '--storagectl', 'SATA Controller', # one of `SATA Controller` or `SCSI Controller` or `IDE Controller`;
        #                                      # obtain the right name using: vboxmanage showvminfo
           '--port', 2,     # port of storage controller. Note that port #0 is for 1st hard disk, so start numbering from 1.
           '--device', 0,   # the device number inside given port (usually is #0)
           '--type', 'hdd',
           '--medium', second_disk] # path to our VDI image
      end

      end # srv.vm.provider virtualbox

      # Provision the VM with Ansible if enabled in machines.yml
      if machine['provision'] != nil
        srv.vm.provision 'ansible' do |ansible|
          ansible.playbook = machine['provision']
          ansible.verbose = true
        end # srv.vm.provision
      end # if machine['provision']
    end # config.vm.define
  end # machines.each
end # Vagrant.configure
