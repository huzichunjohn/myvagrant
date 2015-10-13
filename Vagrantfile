# -*- mode: ruby -*-
# vi: set ft=ruby :

nodes = {
  # prefix     => [count, ip_start, additional_disk_number]
  'controller' => [1, 11, 0],
  'network' => [1, 21, 0],
  'compute' => [1, 31, 0],
  'cinder' => [1, 41, 1],
  'swift' => [2, 51, 2],
}

Vagrant.configure(2) do |config|
  config.vm.box = "hashicorp/precise64"

  nodes.each do |prefix, (count, ip_start, additional_disk_number)|
    count.times do |i|
      hostname = "%s%02d" % [prefix, (i+1)]

      config.vm.define "#{hostname}" do |box|
        box.vm.hostname = "#{hostname}.huzichun.com"
        box.vm.network :private_network, ip: "10.0.0.#{ip_start+i}", :network => "255.255.255.0" 
     
        if prefix == "network"
          box.vm.network :private_network, ip: "10.0.1.#{ip_start+i}", :network => "255.255.255.0"
          box.vm.network :public_network, bridge: "en0: Wi-Fi (AirPort)"
        elsif prefix == "compute"
          box.vm.network :private_network, ip: "10.0.1.#{ip_start+i}", :network => "255.255.255.0"
          box.vm.network :private_network, ip: "10.0.2.#{ip_start+i}", :network => "255.255.255.0"
        elsif prefix == "cinder"
          box.vm.network :private_network, ip: "10.0.2.#{ip_start+i}", :network => "255.255.255.0"
        elsif prefix == "swift"
          box.vm.network :private_network, ip: "10.0.2.#{ip_start+i}", :network => "255.255.255.0"
        end 

        box.vm.provider :virtualbox do |vb|
          vb.customize ["modifyvm", :id, "--natnet1", "192.168.200.0/24"]
          # customize the cpus and memory sizes.
          if prefix == "controller"
            vb.customize ["modifyvm", :id, "--memory", 2048]
            vb.customize ["modifyvm", :id, "--cpus", 1]
          elsif prefix == "network"
            vb.customize ["modifyvm", :id, "--memory", 512]
            vb.customize ["modifyvm", :id, "--cpus", 1]
          elsif prefix == "compute"
            vb.customize ["modifyvm", :id, "--memory", 2048]
            vb.customize ["modifyvm", :id, "--cpus", 2]
          elsif prefix == "cinder"
            vb.customize ["modifyvm", :id, "--memory", 512]
            vb.customize ["modifyvm", :id, "--cpus", 1]
          elsif prefix == "swift"
            vb.customize ["modifyvm", :id, "--memory", 1024]
            vb.customize ["modifyvm", :id, "--cpus", 1]
          end

          dir = File.join(File.dirname(__FILE__), "vagrant-additional-disk")
          unless File.directory?(dir)
            Dir.mkdir dir
          end

          if additional_disk_number > 0
            disk_labels = 'b'..'z'
            disk_labels.to_a.first(additional_disk_number).each_with_index do |disk, i|
              file_to_disk = "#{dir}/#{hostname}-sd#{disk}.vdi"
              unless File.exist?(file_to_disk)
                vb.customize ["createhd", "--filename", file_to_disk, "--size", 20 * 1024]
              end
              vb.customize ["storageattach", :id, "--storagectl", "SATA Controller", "--port", (i+1).to_s, "--device", 0, "--type", "hdd", "--medium", file_to_disk]
            end
          end
        end
      end
    end
  end

  config.vm.provision :ansible do |ansible|
    ansible.playbook = "playbook.yml"
  end
end
