Vagrant.configure("2") do |config|
  config.vm.define "dc01" do |dc01|
    dc01.vm.box= "windows-server-2016-dc"
    # Admin user name and password
    dc01.winrm.username = "vagrant"
    dc01.winrm.password = "vagrant"
    dc01.vm.guest = :windows
    dc01.windows.halt_timeout = 16
    dc01.vm.network :forwarded_port, guest: 3389, host: 3389, id: "rdp", auto_correct: true
    dc01.vm.base_address = "192.168.177.5"  
    dc01.vm.base_mac = "A0B0C0D0E0F0"
    dc01.vm.network "private_network", ip: "10.1.1.40", gateway: "10.1.1.1"
    # vagrant auto configures ethernet 2 as it's main adapter, then ethernet, then ethernet 3.....
    # I'm not sure why and after many hours of trying to auto configure ethernet 2,
    # I opted to come up with this hack solution.
    # We just don't touch ethernet 2 and configure alternative ip's that are static
    dc01.vm.provision "shell", inline: <<-SHELL
      netsh.exe int ip set address "Ethernet" static 10.1.1.40 255.255.255.0 10.1.1.1
      netsh.exe int ip show addresses
    SHELL
    dc01.vm.provision "shell",  path: "../scripts/current/provision-ous.ps1"
    dc01.vm.provision "shell", path: "../scripts/current/provision-computers.ps1"
    dc01.vm.provision "shell", path: "../scripts/current/provision-users.ps1"
    dc01.vm.provision "shell", privileged: "true", path: '../scripts/current/enable-rdp.bat'
    dc01.vm.provision "shell", privileged: "true", path: "../scripts/current/provision-gpos.ps1"
    dc01.vm.provider :vmware_desktop do |v, override|
      v.gui = true
    end
  end
  config.vm.define "srv01" do |srv01|
    srv01.vm.box = "windows-server-2016"
    srv01.vm.network :forwarded_port, guest: 5985, host: 5986, id: "winrm", auto_correct: true
    srv01.vm.network :forwarded_port, guest: 3389, host: 3390, id: "rdp", auto_correct: true
    # this is the network interface that vagrant uses
    srv01.vm.base_address = "192.168.177.10"  
    srv01.vm.base_mac = "A0B0C0D0E0F1"

    srv01.vm.communicator = "winrm"
  
    # Admin user name and password
    srv01.winrm.username =  'Administrator'
    srv01.winrm.password = "vagrant"

    srv01.vm.guest = :windows
    srv01.windows.halt_timeout = 15
    # this is a second network interface that we configure with the inline provisioner script
    srv01.vm.network "private_network", ip: "10.1.1.41", gateway: "10.1.1.1"
    srv01.vm.provision "shell", inline: <<-SHELL
      netsh.exe int ip set address "Ethernet" static 10.1.1.41 255.255.255.0 10.1.1.1
      netsh int ip set dns "Ethernet" static 10.1.1.40
      netsh.exe int ip show addresses
    SHELL

    srv01.vm.provision "shell", privileged: "true", inline: <<-SHELL
      echo 'Renaming the computer to: srv01'
      Rename-Computer -newname srv01
    SHELL
    srv01.vm.provision :reload 
    
    srv01.vm.provision "shell", privileged: "true", path: '../scripts/current/join-domain.ps1'

    srv01.vm.provision :reload 

    srv01.vm.provision "shell", privileged: "true", path: '../scripts/current/enable-rdp.bat'

    srv01.vm.provider :vmware_desktop do |v, override|
      v.gui = true
    end
  end
  config.vm.define "wrk01" do |wrk01|
    wrk01.vm.box = "windows-server-2016"
    wrk01.vm.network :forwarded_port, guest: 5985, host: 5987, id: "winrm", auto_correct: true
    wrk01.vm.network :forwarded_port, guest: 3389, host: 3391, id: "rdp", auto_correct: true
    # this is the network interface that vagrant uses
    wrk01.vm.base_address = "192.168.177.11"  
    wrk01.vm.base_mac = "A0B0C0D0E0F2"
    
    #wrk01.vm.network "private_network", ip: "10.1.1.42", gateway: "10.1.1.1"

    wrk01.vm.communicator = "winrm"
  
    # Admin user name and password
    wrk01.winrm.username =  'Administrator'
    wrk01.winrm.password = "vagrant"

    wrk01.vm.guest = :windows
    wrk01.windows.halt_timeout = 15
    # this is a second network interface that we configure with the inline provisioner script
    wrk01.vm.network "private_network", ip: "10.1.1.42", gateway: "10.1.1.1"
    wrk01.vm.provision "shell", inline: <<-SHELL
      netsh.exe int ip set address "Ethernet" static 10.1.1.42 255.255.255.0 10.1.1.1
      netsh int ip set dns "Ethernet" static 10.1.1.40
      netsh.exe int ip show addresses
    SHELL

    wrk01.vm.provision "shell", privileged: "true", inline: <<-SHELL
      echo 'Renaming the computer to: wrk01'
      Rename-Computer -newname wrk01
    SHELL
    
    wrk01.vm.provision :reload 
    
    wrk01.vm.provision "shell", privileged: "true", path: '../scripts/current/join-domain.ps1'

    wrk01.vm.provision :reload 

    wrk01.vm.provision "shell", privileged: "true", path: '../scripts/current/enable-rdp.bat'

    wrk01.vm.provider :vmware_desktop do |v, override|
      v.gui = true
    end
  end
end
