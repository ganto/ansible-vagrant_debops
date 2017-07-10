## [![DebOps project](http://debops.org/images/debops-small.png)](http://debops.org) vagrant_debops

[![Ansible Galaxy](http://img.shields.io/badge/galaxy-ganto.vagrant_debops-660198.svg?style=flat)](https://galaxy.ansible.com/ganto/vagrant_debops/)
[![Platforms](https://img.shields.io/badge/platform-debian%7Cubuntu-lightgrey.svg?style=flat)](#)

`vagrant_debops` is an Ansible role which can be used with the
[Vagrant](https://www.vagrantup.com) Ansible provisioner to prepare machines
(VMs, Containers) for [DebOps](http://debops.org/). It mainly wraps the
[debops.debops](http://docs.debops.org/en/latest/ansible/roles/ansible-debops/docs/index.html)
role and makes sure the SSH trust in a multi-machine environment is properly
setup.


### Getting Started

#### Requirements

This role is mainly intended to be run with the Ansible provisioner in Vagrant.
Therefore it requires Vagrant (>=1.8) and Ansible (>=2.0) to be installed on
the host. Further it's an advantage if you have some basic knowledge about
those tools. Otherwise you may want to first read at least the following
introductions:

* [vagrantup.com: Getting Started](https://www.vagrantup.com/docs/getting-started)
* [ansible.com: Overview - How Ansible works](https://www.ansible.com/how-ansible-works)
* [vagrantup.com: Ansible and Vagrant - Short Introduction](https://www.vagrantup.com/docs/provisioning/ansible_intro.html)


#### Usage

To setup DebOps in a Vagrant machine follow these steps:

1. Initialize your Vagrant project with a Debian- or Ubuntu-based box. E.g.:

        vagrant init debian/jessie64

2. Create a minimal playbook to run the `ganto.vagrant_debops` role. E.g. in
   `vagrant_debops.yml`:

        - name: Setup DebOps via Vagrant
          hosts: all
          become: True

          roles:
            - role: ganto.vagrant_debops

3. Create a file where you list the Ansible roles you want to use with the Vagrant
   provisioner. With the playbook above you only need `ganto.vagrant_debops`:

        echo ganto.vagrant_debops > requirements.txt

4. Add the Ansible provisioner definition to the project's `Vagrantfile`:

        config.vm.provision "ansible" do |ansible|
          ansible.galaxy_role_file = "requirements.txt"
          ansible.playbook = "vagrant_debops.yml"
          ansible.groups = {
            "debops_all_hosts" => [ "default" ]
          }
        end

5. Run:

        vagrant up
        vagrant ssh

After the machine was successfully provisioned, DebOps with all the roles
and playbooks is setup and a dummy DebOps project was created in
`~vagrant/vagrant-debops`. You can then start using `debops` from within
the machine as described in
[DebOps: Getting started](https://docs.debops.org/en/latest/debops/docs/getting-started.html)


#### Dependencies

This role has the following dependencies to other Ansible Galaxy roles:

* [debops.apt](https://galaxy.ansible.com/debops/apt)
* [debops.apt_preferences](https://galaxy.ansible.com/debops/apt_preferences)
* [debops.debops](https://galaxy.ansible.com/debops/debops)

They will be automatically pulled in when installing `ganto.vagrant_debops`
via `galaxy_role_file` option in the `Vagrantfile` as described above.


#### Multi-machine Vagrant setup

`vagrant_debops` also works and even makes more fun in a Vagrant multi-machine
setup. This means that you can bootstrap a cluster of hosts via Vagrant and
then manage them via DebOps. To do so, follow the official
[multi-machine](https://www.vagrantup.com/docs/multi-machine/index.html)
instructions and additionally specify which machine should be the DebOps
master by adding it to the `[debops_master]` Ansible host group. In your
`Vagrantfile` you would then have something like this:

      (1..3).each do |machine_id|
        config.vm.define "host#{machine_id}"
      end

      config.vm.provision "ansible" do |ansible|
        ansible.galaxy_role_file = "requirements.txt"
        ansible.playbook = "vagrant_debops.yml"
        ansible.groups = {
          "debops_master" => [ "host1" ],
          "debops_all_hosts" => [ "host1", "host2", "host3" ]
        }
      end

This will setup `debops` on 'host1' and configure the SSH trust for 'host1'
to 'host2' and 'host1' to 'host3' so that Ansible from 'host1' can access
'host2' and 'host3' without any further configuration.


#### Examples

You can find a number of example Vagrant projects using the `vagrant_debops`
role in my [vagrant-projects](https://github.com/ganto/vagrant-projects) GitHub
repository.


### Customization

#### How to set Ansible variables via Vagrantfile

Variables used when provisioning DebOps must be set via Vagrant `extra_vars`
option. Like this you can customize
[apt](https://docs.debops.org/en/latest/ansible/roles/ansible-apt/docs/defaults.html)
repositories, `apt` package manager
[preferences](https://docs.debops.org/en/latest/ansible/roles/ansible-apt_preferences/docs/defaults.html),
any [DebOps](https://docs.debops.org/en/latest/ansible/roles/ansible-debops/docs/defaults.html)
settings or the `vagrant_debops` role behaviour itself (see
[Role Variables](#role-variables) below). E.g.:

        config.vm.provision "ansible" do |ansible|
          [...]
          ansible.extra_vars = {
            apt__mirrors: [ 'http://mirror.switch.ch/ftp/mirror/debian/' ],
            vagrant_debops__ssh_keypair: {
              private_key_file: '~/.ssh/id_rsa',
              public_key_file: '~/.ssh/id_rsa.pub'
            }
          }
        end

Before `ansible` is called, Vagrant will generate an Ansible inventory file
from the information provided in the `Vagrantfile`. By default the
`vagrant_debops` role will copy this inventory to the DebOps machine so
it can be also used with DebOps. Variables which should be available for
`debops` therefore must be saved to this inventory file. Vagrant will do this
if they are specified as group variables. E.g.:

        config.vm.provision "ansible" do |ansible|
		ansible.playbook = "vagrant_debops.yml"
          [...]
          ansible.groups = {
            "debops_all_hosts" => [ "default" ],
            "debops_all_hosts:vars" => {
               "apt__mirrors" => "[ 'http://mirror.switch.ch/ftp/mirror/debian/' ]",
               "dhparam__bits" => "[ '1024' ]"
            }
          }
        end


#### Role Variables

##### `vagrant_debops__ssh_keypair`

Install custom SSH keypair to the Vagrant machine which then will be used to
authenticate the DebOps user on different machines. This is especially useful
if you want to test DebOps playbook runs with different Ansible or role
versions against an existing infrastructure.

This is a dictionary variable which support the following configuration keys:

* `private_key_file`: Required. SSH private key file.

* `public_key_file`: Optional. SSH public key file. If this parameter is
  omitted, the public key will be generated from the private key. However,
  this only works if the private key is not pass-phrase protected.

Default value: `{ private_key_file: '{{ ansible_ssh_private_key_file }}' }`


##### `vagrant_debops__ssh_user_home`

Home directory of Vagrant SSH user.

Default value: `'{{ "/root" if ansible_ssh_user == "root" else "/home/" + ansible_ssh_user }}'`


##### `vagrant_debops__ssh_privkey_name`

Target name of the uploaded SSH private key in the machine. By default it
will name the key according to the machine it can connect to. This allows
to manage a Vagrant environment with multiple machines using the same keys
as Vagrant itself. If a custom private key is given, use the default name
(`id_rsa`).

Default value: `'{{ inventory_hostname if vagrant_debops__ssh_keypair["private_key_file"] == ansible_ssh_private_key_file else "id_rsa" }}'`


##### `vagrant_debops__debops__install_systemwide`

Configure the `debops.debops` role to install the DebOps playbooks and roles
system wide.

Default value: `False`


##### `vagrant_debops__debops__update_method`

Method how the `debops.debops` role should download playbooks and roles.
Check [debops__update_method](http://docs.debops.org/en/latest/ansible/roles/ansible-debops/docs/defaults.html#envvar-debops__update_method)
for more information.

Default value: `'sync'`


##### `vagrant_debops__debops__project_name`

Initialize new DebOps project with the given name via `debops.debops` role.
To disable project initialization, set to `False`.

Default value: `'vagrant-debops'`


##### `vagrant_debops__upload_inventory`

Upload inventory generated by Vagrant Ansible provisioner to VM. This will
allow to configure the DebOps inventory completely in the `Vagrantfile`.

Default value: `True`


##### `vagrant_debops__inventory_vars`

Global variables which will be added to the inventory after uploading. This
requires `vagrant_debops__upload_inventory` to be `True`.

Default value:
```
  # Set Vagrant host as Ansible controller to avoid blocking `vagrant ssh`
  core__ansible_controllers: [ '{{ ansible_env["SSH_CLIENT"].split(" ")[0] }}/32' ]
  ferm__ansible_controllers: [ '{{ ansible_env["SSH_CLIENT"].split(" ")[0] }}/32' ]
  tcpwrappers__ansible_controllers: [ '{{ ansible_env["SSH_CLIENT"].split(" ")[0] }}/32' ]
```


### Authors and license

The `vagrant_debops` role was written by:

- [Reto Gantenbein](https://linuxmonk.ch/) | [e-mail](mailto:reto.gantenbein@linuxmonk.ch) | [GitHub](https://github.com/ganto)

License: [GPLv3](https://tldrlegal.com/license/gnu-general-public-license-v3-%28gpl-3%29)
