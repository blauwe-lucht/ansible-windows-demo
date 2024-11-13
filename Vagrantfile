Vagrant.configure("2") do |config|
    
    config.vm.define "acs" do |acs|
        acs.vm.box = "almalinux/9"
        acs.vm.hostname = "acs"

        # We're using a private network which will result in a 'host-only' adapter,
        # which will make the VM accessible directly from the host machine.
        # We specify the IP-address so we can easily refer to each machine in Ansible.
        acs.vm.network "private_network", ip: "192.168.9.26"

        # We're going to use Visual Studio Code Remote SSH to connect to the machines,
        # so we need a way to make the SSH ports predictable (they're not by default).
        # So we explicitly specify the local SSH port for forwarding.
        # Notice that it is derived from the last two numbers of the IP address.
        # See https://realguess.net/2015/10/06/overriding-the-default-forwarded-ssh-port-in-vagrant/
        acs.vm.network "forwarded_port", id: "ssh", guest: 22, host: 2926
        
        # Make sure all sensitive info is only readable by user (SSH requires this
        # when using private keys to connect to a VM).
        acs.vm.synced_folder ".", "/vagrant", mount_options: ["dmode=700,fmode=600"]

        # Nginx needs to be able to access files and directories in ansible/files/packages,
        # so share that with default permissions.
        acs.vm.synced_folder "ansible/files/packages", "/opt/ansible-packages"

        # Do the initial setup of the ACS (Ansible Control Server) with a script.
        acs.vm.provision "shell" do |shell|
            shell.inline = <<-SHELL
                set -euxo pipefail

                echo Installing prerequisites for Docker...
                dnf install -y dnf-plugins-core

                echo Adding Docker repository...
                dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

                echo Installing Docker...
                dnf install -y docker-ce docker-ce-cli containerd.io

                echo Starting and enabling Docker...
                systemctl start docker
                systemctl enable docker

                if [ ! "$(docker ps -q -f name=nginx)" ]; then
                    echo Starting Nginx...
                    docker run --name lighttpd -d -p 80:80 \
                        -v /opt/ansible-packages:/var/www/html:ro \
                        --restart unless-stopped \
                        jitesoft/lighttpd
                fi
                    
                # Pip prefers to run in a virtual environment, so we create one for the vagrant user
                # and use that pip to install ansible and ansible-lint.
                sudo -u vagrant bash -c '
                    set -euxo pipefail

                    python -m venv /home/vagrant/venv-ansible
                    . /home/vagrant/venv-ansible/bin/activate
                    pip install --upgrade pip
                    pip install ansible ansible-lint pywinrm

                    # Make sure we always use ansible from the virtualenv.
                    echo ". ~/venv-ansible/bin/activate" >> ~/.bashrc
                '
            SHELL
        end
    end

    config.vm.define "node1" do |node1|
        node1.vm.box = "gusztavvargadr/windows-server-2022-standard"
        node1.vm.hostname = "node1"
        node1.vm.network "private_network", ip: "192.168.9.27"
        node1.vm.network "forwarded_port", id: "ssh", guest: 22, host: 2927
        node1.vm.provider "virtualbox" do |vb|
            vb.memory = "3072"  # 3 GB of RAM
            vb.cpus = 2
        end
    end

    config.vm.define "node2" do |node2|
        node2.vm.box = "gusztavvargadr/windows-server-2022-standard"
        node2.vm.hostname = "node2"
        node2.vm.network "private_network", ip: "192.168.9.28"
        node2.vm.network "forwarded_port", id: "ssh", guest: 22, host: 2928
        node2.vm.provider "virtualbox" do |vb|
            vb.memory = "3072"
            vb.cpus = 2
        end
    end
end
