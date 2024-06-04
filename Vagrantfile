# -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {
  :otuslinux => {
        :box_name => "ubuntu",
        :vm_name => "zfsvm",
        :ip_addr => '192.168.56.101',		
  },
}
Vagrant.configure("2") do |config|
      config.vm.define "boxname" do |box|
          box.vm.box = "ubuntu"
          box.vm.host_name = "zfsvm"
#          box.vm.synced_folder ".", "/home/pt/vagrant/zfs"
          #box.vm.network "forwarded_port", guest: 3260, host: 3260+offset
          box.vm.network "private_network", ip: '192.168.56.101'
          box.vm.provider :virtualbox do |vb|
          vb.memory = 1024
          vb.cpus = 1
       end
 #        box.vm.disk :disk, size: "1GB", name: "extra_storage"
	  (0..8).each do |i|
 	     box.vm.disk :disk, size: "200MB", name: "disk-#{i}"
#    	end
	end
 	  box.vm.provision "shell", inline: <<-SHELL
	      mkdir -p ~root/.ssh
              cp ~vagrant/.ssh/auth* ~root/.ssh
              sudo sed -i 's/\#PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
              systemctl restart sshd	
              sudo apt install -y zfsutils-linux
  	  SHELL
#      end
  end
end