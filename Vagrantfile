Vagrant.configure(2) do |config|
  config.vm.box = "ubuntu/vivid64"
  config.vm.box_check_update = false
  config.vm.hostname = "box"

  config.vm.provider "virtualbox" do |vb|
    vb.name = "box-ubuntu"
    vb.memory = 2048
    vb.cpus = 2

    vb.customize ["modifyvm", :id, "--accelerate3d", "off"]
    vb.customize ["modifyvm", :id, "--vram", "256"]
    vb.customize ["modifyvm", :id, "--acpi", "on"]
    vb.customize ["modifyvm", :id, "--ioapic", "on"]
  end

  config.vm.provision "shell", inline: <<-SHELL.gsub(/^ +/, '')
    echo "Update package repositories..."
    sudo apt-get update --fix-missing #>/dev/null 2>&1
    #sudo apt-get -y upgrade && apt-get -y autoremove #>/dev/null 2>&1

    echo "Install basic packages..."
    sudo apt-get install -y git vim curl wget whois unzip virtualbox-guest-* #>/dev/null 2>&1

    echo "System configuration..."
    sudo timedatectl set-timezone Europe/Rome
    sudo update-locale LANG=en_US.UTF-8 LC_ALL=en_US.UTF-8
    echo "%sudo ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers

    echo "Create custom user"
    if ! id -u matteo >/dev/null 2>&1; then
        sudo useradd \
          --create-home \
          --gid users \
          --groups sudo \
          --comment "Matteo Pierangeli" \
          --password Do/ZZhJZV/pH2 \
          --shell /bin/bash \
          matteo
    fi

    echo "Install the latest Node version using NVM"
    sudo -iu matteo <<NODEJS
      if [ ! -d ~/.nvm ]; then
          wget -qO- https://raw.githubusercontent.com/creationix/nvm/v0.29.0/install.sh \
            | bash 2>&1
      fi
      source ~/.nvm/nvm.sh
      nvm install node 2>/dev/null
      nvm alias default node
    NODEJS

    echo "Install Docker"
    sudo apt-get install -y docker.io >/dev/null 2>&1
    sudo usermod -aG docker matteo

    echo "Done. Yo man!"
  SHELL
end
