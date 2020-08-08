# -*- mode: ruby -*-
# vim: set ft=ruby :

disks_dir = ENV['HOME']  + '/VirtualBox VMs/'

MACHINES = {
  :otuslinux => {
    :box_name => "centos/8",
    :ip_addr => '192.168.11.101',
    :disks => {
      :sata1 => {
        :dfile => disks_dir + './sata1.vdi',
        :size => 250,
        :port => 1
      },
      :sata2 => {
        :dfile => disks_dir + './sata2.vdi',
        :size => 250, # Megabytes
        :port => 2
      },
      :sata3 => {
        :dfile => disks_dir + './sata3.vdi',
        :size => 250,
        :port => 3
      },
      :sata4 => {
        :dfile => disks_dir + './sata4.vdi',
        :size => 250, # Megabytes
        :port => 4
      },
      :sata5 => {
        :dfile => disks_dir + './sata5.vdi',
        :size => 250, # Megabytes
        :port => 5
      },
    },
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

        # create raid
        mdadm -q --zero-superblock --force /dev/sd{b,c,d,e,f}
        mdadm -q --create --verbose /dev/md0 --level=10 --raid-devices=4 /dev/sd[bcde] --spare-devices=1 /dev/sdf
        mdadm --detail --scan --verbose
        echo "DEVICE partitions" > /etc/mdadm.conf
        mdadm --detail --scan --verbose | awk '/ARRAY/ {print}' >> /etc/mdadm.conf

        # create partitions
        parted -s /dev/md0 mklabel gpt
        for i in $(seq 0 20 80)
          do parted /dev/md0 mkpart primary xfs $i% $(($i + 20))%
        done

        # create fs and mount partitions
        for i in $(seq 1 5)
          do mkfs.xfs -q /dev/md0p$i
        done
        mkdir -p /raid/part{1,2,3,4,5}
        for i in $(seq 1 5); do mount /dev/md0p$i /raid/part$i; done
        
        echo "Provisioning complete!"
      SHELL
    end
  end
end
