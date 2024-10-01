VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
    config.vm.box = "gyptazy/ubuntu22.04-arm64"
    config.vm.hostname = "ansible.local"
    config.vm.provider "vmware_desktop" do |vm|
        vm.memory = 1024
    end

    config.vm.network "private_network", ip: "192.168.33.46"
    config.vm.synced_folder ".", "/vagrant", type: "rsync", rsync_exclude: [".git/"]

    config.vm.provision "shell", inline: <<-SHELL
        export DEBIAN_FRONTEND=noninteractive
        sudo apt -y update
        sudo apt install -y ca-certificates apt-transport-https
        sudo apt install -y software-properties-common curl
        sudo apt install -y python3-pip python-is-python3
        sudo add-apt-repository --yes --update ppa:ansible/ansible
        sudo apt install -y ansible
        sudo pip3 install docker

    SHELL
end
