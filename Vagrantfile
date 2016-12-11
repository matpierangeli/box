Vagrant.configure(2) do |config|
  config.vm.box = "ubuntu/trusty64"
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

  config.vm.network "private_network", type: "dhcp", bridge: "en1: Wi-Fi (AirPort)"
  config.vm.synced_folder "~/Development", "/mnt/host-dev"

  CONSOLE_PREFIX = "[box]->"
  config.vm.provision "shell", inline: <<-SHELL.gsub(/^ +/, '')
    echo "#{CONSOLE_PREFIX} Update package repositories..."
    sudo apt-get update --fix-missing >/dev/null 2>&1
    sudo apt-get -y upgrade && apt-get -y autoremove >/dev/null 2>&1

    echo "#{CONSOLE_PREFIX} Install basic packages..."
    sudo apt-get install -y git vim curl wget whois unzip tree autojump apt-show-versions linux-kernel-headers >/dev/null 2>&1

    echo "#{CONSOLE_PREFIX} System configuration..."
    sudo timedatectl set-timezone Europe/Rome
    sudo update-locale LANG=en_US.UTF-8 LC_ALL=en_US.UTF-8
    echo "%sudo ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers

    echo "#{CONSOLE_PREFIX} Create matteo user..."
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

    echo "#{CONSOLE_PREFIX} Donwload colors..."
    sudo -iu matteo <<DIRCOLORS
      if [ ! -d ~/.dircolors-solarized ]; then
        git clone https://github.com/seebi/dircolors-solarized.git ~/.dircolors-solarized >/dev/null 2>&1
      fi
    DIRCOLORS

    echo "#{CONSOLE_PREFIX} Install dotfiles..."
    sudo -iu matteo <<DOTFILES
      [[ ! -d ~/.dotfiles ]] && git clone https://github.com/matpierangeli/dotfiles.git --branch bash --single-branch ~/.dotfiles >/dev/null 2>&1
      bash ~/.dotfiles/install.sh
      source ~/.profile
    DOTFILES

    echo "#{CONSOLE_PREFIX} Setup dev folder..."
    sudo -iu matteo <<DEVFOLDER
      [[ ! -h ~/dev ]] && ln -s /mnt/host-dev ~/dev
    DEVFOLDER

    echo "#{CONSOLE_PREFIX} Install the latest NodeJS version using NVM..."
    sudo -iu matteo <<NODEJS
      if [ ! -d ~/.nvm ]; then
          wget -qO- https://raw.githubusercontent.com/creationix/nvm/master/install.sh | bash
      fi
      source ~/.nvm/nvm.sh
      nvm install node >/dev/null 2>&1
      nvm alias default node >/dev/null 2>&1
    NODEJS

    echo "#{CONSOLE_PREFIX} Install Docker"
    sudo -i <<DOCKER
        if ! docker version &>/dev/null; then
            wget -qO- https://get.docker.com/ | bash
            sudo usermod -aG docker matteo
        fi
        if ! docker-compose --version &>/dev/null; then
            curl --silent -L https://github.com/docker/compose/releases/download/1.8.0/docker-compose-`uname -s`-`uname -m` \
                    > /usr/local/bin/docker-compose
            chmod +x /usr/local/bin/docker-compose
        fi
    DOCKER

    echo "#{CONSOLE_PREFIX} Done. Yo man!"
  SHELL
end