# -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {
  :otuslinux => {
        :box_name => "centos/7",
        :ip_addr => '192.168.11.101',
	:disks => {
		:sata1 => {
			:dfile => './sata1.vdi',
			:size => 250,
			:port => 1
		},
		:sata2 => {
                        :dfile => './sata2.vdi',
                        :size => 250, # Megabytes
			:port => 2
		},
                :sata3 => {
                        :dfile => './sata3.vdi',
                        :size => 250,
                        :port => 3
                },
                :sata4 => {
                        :dfile => './sata4.vdi',
                        :size => 250, # Megabytes
                        :port => 4
                },
		:sata5 => {
                        :dfile => './sata5.vdi',
                        :size => 250,
                        :port => 5
                },
		:sata6 => {
                        :dfile => './sata6.vdi',
                        :size => 250,
                        :port => 6
                },



	}

		
  },
}

Vagrant.configure("2") do |config|

  MACHINES.each do |boxname, boxconfig|

      config.vm.define boxname do |box|

          box.vm.box = boxconfig[:box_name]
          box.vm.host_name = boxname.to_s

          #box.vm.network "forwarded_port", guest: 3260, host: 3260+offset

          box.vm.network "private_network", ip: boxconfig[:ip_addr]

          box.vm.provider :virtualbox do |vb|
            	  vb.customize ["modifyvm", :id, "--memory", "1024"]
                  needsController = false
		  boxconfig[:disks].each do |dname, dconf|
			  unless File.exist?(dconf[:dfile])
				vb.customize ['createhd', '--filename', dconf[:dfile], '--variant', 'Fixed', '--size', dconf[:size]]
                                needsController =  true
                          end

		  end
                  if needsController == true
                     vb.customize ["storagectl", :id, "--name", "SATA", "--add", "sata" ]
                     boxconfig[:disks].each do |dname, dconf|
                         vb.customize ['storageattach', :id,  '--storagectl', 'SATA', '--port', dconf[:port], '--device', 0, '--type', 'hdd', '--medium', dconf[:dfile]]
                     end
                  end
          end
 	  box.vm.provision "shell", inline: <<-SHELL
	      mkdir -p ~root/.ssh
              cp ~vagrant/.ssh/auth* ~root/.ssh
	      yum install -y mdadm smartmontools hdparm gdisk
	      mdadm --zero-superblock --force /dev/sd{b,c,d,e,f,g}
	      mdadm --create --verbose /dev/md0 -l 6 -n 6 /dev/sd{b,c,d,e,f,g}
	      sudo echo "DEVICE partitions" >> /etc/mdadm.conf
	      sudo mdadm --detail --scan --verbose | awk '/ARRAY/ {print}' >> /etc/mdadm.conf
	      sudo parted -s /dev/md0 mklabel gpt
	      sudo parted /dev/md0 mkpart primary ext4 0% 16%
		sudo parted /dev/md0 mkpart primary ext4 16% 33%
		sudo parted /dev/md0 mkpart primary ext4 33% 49%
		sudo parted /dev/md0 mkpart primary ext4 49% 65%
		sudo parted /dev/md0 mkpart primary ext4 65% 81%
		sudo parted /dev/md0 mkpart primary ext4 81% 100%
		sudo  mkdir -p /raid
		sudo mkdir /raid/part{1,2,3,4,5,6}
		sudo mkfs.ext4 /dev/md0p1
		sudo mkfs.ext4 /dev/md0p2
		sudo mkfs.ext4 /dev/md0p3
		sudo mkfs.ext4 /dev/md0p4
		sudo mkfs.ext4 /dev/md0p5
		sudo mkfs.ext4 /dev/md0p6
  	  	sudo echo "/dev/md0p1 /raid/part1    defaults 0 0" >> /etc/fstab
                sudo echo "/dev/md0p2 /raid/part2    defaults 0 0" >> /etc/fstab
                sudo echo "/dev/md0p3 /raid/part3    defaults 0 0" >> /etc/fstab
                sudo echo "/dev/md0p4 /raid/part4    defaults 0 0" >> /etc/fstab
                sudo echo "/dev/md0p5 /raid/part5    defaults 0 0" >> /etc/fstab
		sudo echo "/dev/md0p6 /raid/part6    defaults 0 0" >> /etc/fstab
	  SHELL

      end
  end
end

