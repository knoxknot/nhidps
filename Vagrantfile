require 'yaml'
vars = YAML.load_file 'variables.yaml'

Vagrant.configure("2") do |config|
  # import box
  config.vm.tap do |vm|
    vm.box = "#{vars['box_name']}"
    vm.box_url = "file://box/#{vars['box_name']}.box"
    vm.box_download_checksum =  File.read("box/checksums/#{vars['checksum_type']}.txt").split.first
    vm.box_download_checksum_type = "#{vars['checksum_type']}"
  end

  # specify ssh settings
  config.ssh.tap do |ssh|
    ssh.verify_host_key = false
	  ssh.forward_agent = false
	  ssh.forward_x11 = false
    ssh.username = "#{vars['user']}"
    ssh.password = "#{vars['user_password']}"
  end

  # specify vm provider settings
  config.vm.provider "virtualbox" do |vb|
    vb.name = "nhidps"
    vb.gui = true
    vb.cpus = "2"
    vb.memory = "2048"
    vb.customize ["modifyvm", :id, "--clipboard-mode", "bidirectional", "--vram", "8"]
    vb.customize ["setextradata", :id, "VBoxInternal2/EfiGopMode", "1"]
  end

  # configure vm 
  config.vm.define "nhidps" do |nhidps|
    nhidps.vm.hostname = "nhidps"
    nhidps.vm.synced_folder ".", "/home/#{vars['user']}/#{vars['box_name']}"
    nhidps.vm.provision "shell", run: "once", inline: <<-SCRIPT
      cat /home/#{vars['user']}/#{vars['box_name']}/nhidps.pub >> /home/#{vars['user']}/.ssh/authorized_keys
      echo "#{vars['user']}:#{vars['user_password']}" | sudo chpasswd
      echo -e "nameserver 1.1.1.3\nnameserver 1.0.0.3" | sudo tee /etc/resolv.conf
      sudo nmcli connection modify "Wired connection 1" ipv4.dns "1.1.1.3 1.0.0.3"
      sudo nmcli connection modify "Wired connection 1" ipv6.method "disabled"
      sudo systemctl restart NetworkManager 
      sudo apt update -y --fix-missing && sudo apt upgrade -y && sudo apt autoremove -y
      echo "root:#{vars['root_password']}" | sudo chpasswd
    SCRIPT
    nhidps.vm.provision :ansible do |ansible|
      ansible.playbook = "ansible/vagrant.yml"
      ansible.extra_vars = {
        ansible_ssh_private_key_file: "nhidps",
        ansible_python_interpreter: "/usr/bin/python3"
      }
      ansible.verbose = "vvvv"
    end
  end 
end
