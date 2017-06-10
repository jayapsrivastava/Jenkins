# -*- mode: ruby -*-
# vi: set ft=ruby :


vmnet_number = 2

#In order to get the host OS
module OS
	def OS.windows?
		(/cygwin|mswin|mingw|bccwin|wince|emx/ =~ RUBY_PLATFORM) != nil
	end

	def OS.mac?
		(/darwin/ =~ RUBY_PLATFORM) != nil
	end

	def OS.unix?
		!OS.windows?
	end

	def OS.linux?
		OS.unix? and not OS.mac?
	end
end

if OS.mac?
	vdiskmanager = '/Applications/VMware\ Fusion.app/Contents/Library/vmware-vdiskmanager'
else
	vdiskmanager = '/usr/bin/vmware-vdiskmanager'
end		

#Installing required plugins
#re-exec if new plugin was installed.
if ARGV[0] == "up"
	if ARGV[1] == "--provider=vmware_fusion"
		required_plugins = %w(vagrant-scp vagrant-share vagrant-vmware-fusion)

		plugins_to_install = required_plugins.select { |plugin| not Vagrant.has_plugin? plugin }
		if not plugins_to_install.empty?
			puts "Installing plugins: #{plugins_to_install.join(' ')}"
			if system "vagrant plugin install #{plugins_to_install.join(' ')}"
				exec "vagrant #{ARGV.join(' ')}"
			else
				abort "Installation of one or more plugins has failed. Aborting."
			end
		end
	elsif ARGV[1] == "--provider=vmware_workstation"
		required_plugins = %w(vagrant-scp vagrant-share vagrant-vmware-workstation)

		plugins_to_install = required_plugins.select { |plugin| not Vagrant.has_plugin? plugin }
		if not plugins_to_install.empty?
			puts "Installing plugins: #{plugins_to_install.join(' ')}"
			if system "vagrant plugin install #{plugins_to_install.join(' ')}"
				exec "vagrant #{ARGV.join(' ')}"
			else
				abort "Installation of one or more plugins has failed. Aborting."
			end
		end
	else
		abort "Usage : vagrant up --provider=[vmware_fusion|vmware_workstation]"
	end
end

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.

Vagrant.configure("2") do |config|

	# 1. VM for Customer Lustre MDS	
	config.vm.define "CS-cl-MDS", primary: true do |camsrv|
		camsrv.vm.box = 'bento/centos-7.3'
		#camsrv.vm.box = "centos/7"
		camsrv.vm.hostname = "CS-cl-MDS"
		camsrv.vm.network "private_network", ip: "10.2.2.71"
		camsrv.vm.synced_folder ".", "/vagrant", type: "rsync"
		camsrv.vm.synced_folder "src/", "/root/src", type: "rsync"
		camsrv.vm.provision "ansible" do |ansible|
			ansible.verbose = "-vvv"
			ansible.limit = "all"
			ansible.playbook = "provisioning/CS-cl-MDS.yml"
			ansible.inventory_path = "provisioning/inventory/inventory"
		end	
		camsrv.vm.provider :vmware_fusion do |vm, override|
			vm.linked_clone = false
			vm.gui = true

			dir = "#{ENV['HOME']}/vagrant-additional-disk"
			file_to_disk = "#{dir}/drive_CS-cl-MDS.vmdk"
			if ARGV[0] == "up"
				unless File.directory?( dir )
					Dir.mkdir dir
				end
				unless File.exists?( file_to_disk )
					`#{vdiskmanager} -c -s 1GB -a lsilogic -t 1 #{file_to_disk}`
				end
			elsif ARGV[0] == "destroy"
				if ARGV[2] == "CS-cl-MDS"
					Dir.entries("#{ENV['HOME']}/vagrant-additional-disk").grep(/^#{file_to_disk}/).each do |fd|
						File.unlink(fd)
						puts "file #{fd}" 
					end
				else
					`rm -rf #{dir}/*`
				end
			end
			vm.vmx["memsize"] = "1024"
			vm.vmx["numvcpus"] = "2"
			vm.vmx['scsi0:1.filename'] = file_to_disk
			vm.vmx['scsi0:1.present']  = 'TRUE'
			vm.vmx['scsi0:1.redo']     = ''
		end
		camsrv.vm.provider :vmware_workstation do |vmw, override|
			vmw.vmx["ethernet0.present"] = "TRUE"
			vmw.vmx["ethernet0.connectionType"] = "nat"
			vmw.vmx["ethernet0.vnet"] = "/dev/vmnet8"
			vmw.vmx["ethernet0.virtualDev"] = "e1000e"
			vmw.vmx["ethernet1.connectionType"] = "custom"
			vmw.vmx["ethernet1.present"] = "TRUE"
			vmw.vmx["ethernet1.vnet"] = "/dev/vmnet#{vmnet_number}"
			vmw.vmx["ethernet1.virtualDev"] = "e1000e"
			vmw.linked_clone = false
			dir = "#{ENV['HOME']}/vagrant-additional-disk"
			file_to_disk = "#{dir}/drive_CS-cl-MDS.vmdk"
			if ARGV[0] == "up"
				unless File.directory?( dir )
					Dir.mkdir dir
				end
				unless File.exists?( file_to_disk )
					`#{vdiskmanager} -c -s 1GB -a lsilogic -t 1 #{file_to_disk}`
				end
			elsif ARGV[0] == "destroy"
				Dir.entries("#{ENV['HOME']}/vagrant-additional-disk").grep(/^#{file_to_disk}/).each do |fd|
					File.unlink(fd)
				end
			end
			vmw.vmx["memsize"] = "1024"
			vmw.vmx["numvcpus"] = "2"		
			vmw.vmx['scsi0:1.filename'] = file_to_disk
			vmw.vmx['scsi0:1.present']  = 'TRUE'
			vmw.vmx['scsi0:1.redo']     = ''
        end

	end

	#2. Another VM for Customer Lustre OSS 

	config.vm.define "CS-cl-OSS", primary: true do |camcli|
		camcli.vm.box = 'bento/centos-7.3' 
		camcli.vm.hostname = "CS-cl-OSS"
		camcli.vm.network "private_network", ip: "10.2.2.62" 
		camcli.vm.synced_folder ".", "/vagrant", type: "rsync"
		camcli.vm.synced_folder "src/", "/root/src", type: "rsync"
		camcli.vm.provision "ansible" do |ansible|
			ansible.verbose = "-vvv"
			#		ansible.limit = "all"
			ansible.playbook = "provisioning/CS-cl-OSS.yml"
			ansible.inventory_path = "provisioning/inventory/inventory"
		end	
		camcli.vm.provider :vmware_fusion do |vm, override|
			vm.linked_clone = false
			vm.gui = true

			dir = "#{ENV['HOME']}/vagrant-additional-disk"
			file_to_disk = "#{dir}/drive_CS-cl-OSS.vmdk"
			if ARGV[0] == "up"
				unless File.directory?( dir )
					Dir.mkdir dir
				end
				unless File.exists?( file_to_disk )
					`#{vdiskmanager} -c -s 1GB -a lsilogic -t 1 #{file_to_disk}`
				end
			elsif ARGV[0] == "destroy"
				`rm -f #{file_to_disk}`
				`rm -rf #{dir}/*`
			end
			vm.vmx["memsize"] = "1024"
			vm.vmx["numvcpus"] = "2"
			vm.vmx['scsi0:1.filename'] = file_to_disk
			vm.vmx['scsi0:1.present']  = 'TRUE'
			vm.vmx['scsi0:1.redo']     = ''
		end
		camcli.vm.provider :vmware_workstation do |vmw, override|
			vmw.vmx["ethernet0.present"] = "TRUE"
			vmw.vmx["ethernet0.connectionType"] = "nat"
			vmw.vmx["ethernet0.vnet"] = "/dev/vmnet8"
			vmw.vmx["ethernet0.virtualDev"] = "e1000e"
			vmw.vmx["ethernet1.connectionType"] = "custom"
			vmw.vmx["ethernet1.present"] = "TRUE"
			vmw.vmx["ethernet1.vnet"] = "/dev/vmnet#{vmnet_number}"
			vmw.vmx["ethernet1.virtualDev"] = "e1000e"
			vmw.linked_clone = false
			dir = "#{ENV['HOME']}/vagrant-additional-disk"
			file_to_disk = "#{dir}/drive_CS-cl-OSS.vmdk"
			if ARGV[0] == "up"
				unless File.directory?( dir )
					Dir.mkdir dir
				end
				unless File.exists?( file_to_disk )
					`#{vdiskmanager} -c -s 1GB -a lsilogic -t 1 #{file_to_disk}`
				end
			elsif ARGV[0] == "destroy"
				`rm -f #{file_to_disk}`
				`rm -rf #{dir}/*`
			end
			vmw.vmx["memsize"] = "1024"
			vmw.vmx["numvcpus"] = "2"		
			vmw.vmx['scsi0:1.filename'] = file_to_disk
			vmw.vmx['scsi0:1.present']  = 'TRUE'
			vmw.vmx['scsi0:1.redo']     = ''
        end

	end
	#3. Another VM for Marfs MDS 
	config.vm.define "CS-MarFS-MDS", primary: true do |cammv|
		cammv.vm.box = 'bento/centos-7.3'
		cammv.vm.hostname = "CS-MarFS-MDS"
		cammv.vm.network "private_network", ip: "10.2.2.63" 
		cammv.vm.synced_folder ".", "/vagrant", type: "rsync"
		cammv.vm.synced_folder "src/", "/root/src", type: "rsync"
		cammv.vm.provision "ansible" do |ansible|
			ansible.limit = "all"
			ansible.verbose = "-vvv"
			ansible.playbook = "provisioning/CS-MarFS-MDS.yml"
			ansible.inventory_path = "provisioning/inventory/inventory"
		end	
		cammv.vm.provider :vmware_fusion do |vm, override|
			vm.linked_clone = false
			vm.gui = true

			dir = "#{ENV['HOME']}/vagrant-additional-disk"
			file_to_disk = "#{dir}/drive_CS-MarFS-MDS.vmdk"
			if ARGV[0] == "up"
				unless File.directory?( dir )
					Dir.mkdir dir
				end
				unless File.exists?( file_to_disk )
					`#{vdiskmanager} -c -s 1GB -a lsilogic -t 1 #{file_to_disk}`
				end
			elsif ARGV[0] == "destroy"
				`rm -f #{file_to_disk}`
				`rm -rf #{dir}/*`
			end
			vm.vmx["memsize"] = "1024"
			vm.vmx["numvcpus"] = "2"
			vm.vmx['scsi0:1.filename'] = file_to_disk
			vm.vmx['scsi0:1.present']  = 'TRUE'
			vm.vmx['scsi0:1.redo']     = ''
		end
		cammv.vm.provider :vmware_workstation do |vmw, override|
			vmw.vmx["ethernet0.present"] = "TRUE"
			vmw.vmx["ethernet0.connectionType"] = "nat"
			vmw.vmx["ethernet0.vnet"] = "/dev/vmnet8"
			vmw.vmx["ethernet0.virtualDev"] = "e1000e"
			vmw.vmx["ethernet1.connectionType"] = "custom"
			vmw.vmx["ethernet1.present"] = "TRUE"
			vmw.vmx["ethernet1.vnet"] = "/dev/vmnet#{vmnet_number}"
			vmw.vmx["ethernet1.virtualDev"] = "e1000e"
			vmw.linked_clone = false
			dir = "#{ENV['HOME']}/vagrant-additional-disk"
			file_to_disk = "#{dir}/drive_CS-MarFS-MDS.vmdk"
			if ARGV[0] == "up"
				unless File.directory?( dir )
					Dir.mkdir dir
				end
				unless File.exists?( file_to_disk )
					`#{vdiskmanager} -c -s 1GB -a lsilogic -t 1 #{file_to_disk}`
				end
			elsif ARGV[0] == "destroy"
				`rm -f #{file_to_disk}`
				`rm -rf #{dir}/*`
			end
			vmw.vmx["memsize"] = "1024"
			vmw.vmx["numvcpus"] = "2"		
			vmw.vmx['scsi0:1.filename'] = file_to_disk
			vmw.vmx['scsi0:1.present']  = 'TRUE'
			vmw.vmx['scsi0:1.redo']     = ''
        end

	end

	# 4. Another VM for MarFS OSS
	config.vm.define "CS-MarFS-OSS", primary: true do |cammv|
		cammv.vm.box = 'bento/centos-7.3' 
		cammv.vm.hostname = "CS-MarFS-OSS"
		cammv.vm.network "private_network", ip: "10.2.2.64" 
		cammv.vm.synced_folder ".", "/vagrant", type: "rsync"
		cammv.vm.synced_folder "src/", "/root/src", type: "rsync"
		cammv.vm.provision "ansible" do |ansible|
			ansible.limit = "all"
			ansible.verbose = "-vvv"
			ansible.playbook = "provisioning/CS-MarFS-OSS.yml"
			ansible.inventory_path = "provisioning/inventory/inventory"
		end	
		cammv.vm.provider :vmware_fusion do |vm, override|
			vm.linked_clone = false
			vm.gui = true

			dir = "#{ENV['HOME']}/vagrant-additional-disk"
			file_to_disk = "#{dir}/drive_CS-MarFS-OSS.vmdk"
			if ARGV[0] == "up"
				unless File.directory?( dir )
					Dir.mkdir dir
				end
				unless File.exists?( file_to_disk )
					`#{vdiskmanager} -c -s 1GB -a lsilogic -t 1 #{file_to_disk}`
				end
			elsif ARGV[0] == "destroy"
				`rm -f #{file_to_disk}`
				`rm -rf #{dir}/*`
			end
			vm.vmx["memsize"] = "1024"
			vm.vmx["numvcpus"] = "2"
			vm.vmx['scsi0:1.filename'] = file_to_disk
			vm.vmx['scsi0:1.present']  = 'TRUE'
			vm.vmx['scsi0:1.redo']     = ''
		end
		cammv.vm.provider :vmware_workstation do |vmw, override|
			vmw.vmx["ethernet0.present"] = "TRUE"
			vmw.vmx["ethernet0.connectionType"] = "nat"
			vmw.vmx["ethernet0.vnet"] = "/dev/vmnet8"
			vmw.vmx["ethernet0.virtualDev"] = "e1000e"
			vmw.vmx["ethernet1.connectionType"] = "custom"
			vmw.vmx["ethernet1.present"] = "TRUE"
			vmw.vmx["ethernet1.vnet"] = "/dev/vmnet#{vmnet_number}"
			vmw.vmx["ethernet1.virtualDev"] = "e1000e"
			vmw.linked_clone = false
			dir = "#{ENV['HOME']}/vagrant-additional-disk"
			file_to_disk = "#{dir}/drive_CS-MarFS-OSS.vmdk"
			if ARGV[0] == "up"
				unless File.directory?( dir )
					Dir.mkdir dir
				end
				unless File.exists?( file_to_disk )
					`#{vdiskmanager} -c -s 1GB -a lsilogic -t 1 #{file_to_disk}`
				end
			elsif ARGV[0] == "destroy"
				`rm -f #{file_to_disk}`
				`rm -rf #{dir}/*`
			end
			vmw.vmx["memsize"] = "1024"
			vmw.vmx["numvcpus"] = "2"		
			vmw.vmx['scsi0:1.filename'] = file_to_disk
			vmw.vmx['scsi0:1.present']  = 'TRUE'
			vmw.vmx['scsi0:1.redo']     = ''
        end

	end

	# 5. Another VM for mover node
	config.vm.define "CS-mv-node", primary: true do |cammv|
		cammv.vm.box = 'bento/centos-7.3' 
		cammv.vm.hostname = "CS-mv-node"
		cammv.vm.network "private_network", ip: "10.2.2.65" 
		cammv.vm.synced_folder ".", "/vagrant", type: "rsync"
		cammv.vm.synced_folder "src/", "/root/src", type: "rsync"
		cammv.vm.provision "ansible" do |ansible|
			#	ansible.limit = "all"
			ansible.verbose = "-vvv"
			ansible.playbook = "provisioning/CS-mv-node.yml"
			ansible.inventory_path = "provisioning/inventory/inventory"
		end	
		cammv.vm.provider :vmware_fusion do |vm, override|
			vm.linked_clone = false
			vm.gui = true

			dir = "#{ENV['HOME']}/vagrant-additional-disk"
			file_to_disk = "#{dir}/drive_CS-mv-node.vmdk"
			if ARGV[0] == "up"
				unless File.directory?( dir )
					Dir.mkdir dir
				end
				unless File.exists?( file_to_disk )
					`#{vdiskmanager} -c -s 1GB -a lsilogic -t 1 #{file_to_disk}`
				end
			elsif ARGV[0] == "destroy"
				`rm -f #{file_to_disk}`
				`rm -rf #{dir}/*`
			end
			vm.vmx["memsize"] = "1024"
			vm.vmx["numvcpus"] = "2"
			vm.vmx['scsi0:1.filename'] = file_to_disk
			vm.vmx['scsi0:1.present']  = 'TRUE'
			vm.vmx['scsi0:1.redo']     = ''
		end
		cammv.vm.provider :vmware_workstation do |vmw, override|
			vmw.vmx["ethernet0.present"] = "TRUE"
			vmw.vmx["ethernet0.connectionType"] = "nat"
			vmw.vmx["ethernet0.vnet"] = "/dev/vmnet8"
			vmw.vmx["ethernet0.virtualDev"] = "e1000e"
			vmw.vmx["ethernet1.connectionType"] = "custom"
			vmw.vmx["ethernet1.present"] = "TRUE"
			vmw.vmx["ethernet1.vnet"] = "/dev/vmnet#{vmnet_number}"
			vmw.vmx["ethernet1.virtualDev"] = "e1000e"
			vmw.linked_clone = false
			dir = "#{ENV['HOME']}/vagrant-additional-disk"
			file_to_disk = "#{dir}/drive_CS-mv-nodes.vmdk"
			if ARGV[0] == "up"
				unless File.directory?( dir )
					Dir.mkdir dir
				end
				unless File.exists?( file_to_disk )
					`#{vdiskmanager} -c -s 1GB -a lsilogic -t 1 #{file_to_disk}`
				end
			elsif ARGV[0] == "destroy"
				`rm -f #{file_to_disk}`
				`rm -rf #{dir}/*`
			end
			vmw.vmx["memsize"] = "1024"
			vmw.vmx["numvcpus"] = "2"		
			vmw.vmx['scsi0:1.filename'] = file_to_disk
			vmw.vmx['scsi0:1.present']  = 'TRUE'
			vmw.vmx['scsi0:1.redo']     = ''
        end

	end


	# Create a forwarded port mapping which allows access to a specific port
	# within the machine from a port on the host machine. In the example below,
	# accessing "localhost:8080" will access port 80 on the guest machine.
	# config.vm.network "forwarded_port", guest: 80, host: 8080

	# Share an additional folder to the guest VM. The first argument is
	# the path on the host to the actual folder. The second argument is
	# the path on the guest to mount the folder. And the optional third
	# argument is a set of non-required options.
	# config.vm.synced_folder "../data", "/vagrant_data"

end
