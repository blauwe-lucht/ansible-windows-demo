# ansible-windows-demo

Demo that shows how to configure Windows machines using Ansible.

## Prerequisites

- [Visual Studio Code](https://code.visualstudio.com/) (vscode)
- [VirtualBox](https://www.virtualbox.org/), tested with version 7.0 (7.1 didn't work)
- [Vagrant](https://www.vagrantup.com/), tested with version 2.4.3

## Usage

1. ```vagrant up```
This will create three VMs: acs (Linux machine with Ansible installed), node1 and node2 (Windows machines).
2. Install the [Remote SSH extension](https://code.visualstudio.com/docs/remote/ssh) in vscode
3. In vscode: F1 -> Remote-SSH: Open SSH Configuration File, select the one in your user directory, add

    ``` ssh
    Host acs
        HostName 192.168.9.26
        User vagrant
        IdentityFile <path to project>/.vagrant/machines/acs/virtualbox/private_key
    ```

4. In vscode left click the connection button in the bottom left (blue or green with something that
looks like ><) or press F1 and select Remote-SSH: Connect to Host. Select host 'acs', select Linux.
Vscode will now install everything that is needed to work with files on the acs. When it's ready, open
the directory /vagrant on the acs. Trust the directory. Install the recommended extensions
([Ansible](https://marketplace.visualstudio.com/items?itemName=redhat.ansible) and
[Ansible Go to Definition](https://marketplace.visualstudio.com/items?itemName=BlauweLucht.ansible-go-to-definition)).
5. Download the [Notepad++ Installer](https://github.com/notepad-plus-plus/notepad-plus-plus/releases/download/v8.7.1/npp.8.7.1.Installer.x64.exe)
to ansible/files/packages.
6. Come up with a good password for [Ansible Vault](https://docs.ansible.com/ansible/latest/vault_guide/vault_encrypting_content.html) and
save it to /home/vagrant/.vault_pass. You can do this with ```echo UltraSecret > /home/vagrant/.vault_pass``` in a terminal window.
7. In vscode open a terminal window and type:

    ``` bash
    cd /vagrant/ansible
    ansible-playbook playbook.yml -v
    ```

8. Wait for a bit and if everything went fine, node1 and node2 will be configured.

## Clean up

In the original vscode instance (so not in the Remote-SSH session) or in a command prompt, go to the
project directory and type:

``` bash
vagrant destroy -f
```
