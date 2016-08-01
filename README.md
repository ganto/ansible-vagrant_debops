## [![DebOps project](http://debops.org/images/debops-small.png)](http://debops.org) vagrant_debops

[![Ansible Galaxy](http://img.shields.io/badge/galaxy-ganto.vagrant_debops-660198.svg?style=flat)](https://galaxy.ansible.com/ganto/vagrant_debops/)
[![Platforms](https://img.shields.io/badge/platform-debian%7Cubuntu-lightgrey.svg?style=flat)](#)

`vagrant_debops` is an Ansible role which can be used with the
[Vagrant](https://www.vagrantup.com) Ansible provisioner to prepare machines
(VMs, Containers) for [DebOps](http://debops.org/). It mainly wraps the
[debops.debops](http://docs.debops.org/en/latest/ansible/roles/ansible-debops/docs/index.html)
role and makes sure the SSH trust in a multi-machine environment is properly
setup.


### Requirements

This role is mainly intended to be run with the Ansible provisioner in Vagrant.
Therefore it requires Vagrant (>=1.8) and Ansible (>=2.0) to be installed on
the host. Further it's an advantage if you have some basic knowledge about
those tools. Otherwise you may want to first read at least the following
introductions:

* [vagrantup.com: Getting Started](https://www.vagrantup.com/docs/getting-started)
* [ansible.com: Overview - How Ansible works](https://www.ansible.com/how-ansible-works)
* [vagrantup.com: Ansible and Vagrant - Short Introduction](https://www.vagrantup.com/docs/provisioning/ansible_intro.html)


### Installation

To install this Ansible role with normal user privileges make sure you define
a role directory within you user home. Add the following line to the
`[defaults]` section of your `~/.ansible.cfg`:

    roles_path = ~/.ansible/roles:/etc/ansible/roles

Then run:

    ansible-galaxy install ganto.vagrant_debops


#### Dependencies

This role has the following dependencies to other Ansible Galaxy roles:

* [debops.debops](https://galaxy.ansible.com/debops/debops)

They will be automatically pulled in when installing `vagrant_debops` as
described above.


### Usage

To setup DebOps in a Vagrant machine follow these steps:

1. Initialize your Vagrant project with a Debian- or Ubuntu-based box. E.g.:

        vagrant init debian/jessie64

2. Copy the provided playbook to the project directory:

        cp ~/.ansible/roles/ganto.vagrant_debops/docs/playbooks/vagrant_debops.yml .

   If you already have a playbook, extend it with:

        - name: Setup DebOps via Vagrant
          hosts: all
          become: True

          roles:
            - role: ganto.vagrant_debops

3. Add the Ansible provisioner definition to the project's `Vagrantfile`:

        config.vm.provision "ansible" do |ansible|
          ansible.playbook = "vagrant_debops.yml"
        end

4. Run:

        vagrant up
        vagrant ssh

After the machine was successfully provisioned, DebOps with all the roles
and playbooks is setup and a dummy DebOps project was created in
`~vagrant/debops_vagrant`.


### Customization

The following role `vagrant_debops` role variables can be used to customize
the role operation.

#### Variables


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


#### How to set Ansible variables via Vagrantfile

The role variables listed above as well as any other role variable from the
`debops.debops` role or any DebOps role can be defined in the `Vagrantfile`.
E.g.:

        config.vm.provision "ansible" do |ansible|
          ansible.playbook = "vagrant_debops.yml"
          ansible.extra_vars = {
            vagrant_debops__ssh_keypair: {
              private_key_file: '~/.ssh/id_rsa',
              public_key_file: '~/.ssh/id_rsa.pub'
            }
          }
        end


### Authors and license

The `vagrant_debops` role was written by:

- [Reto Gantenbein](https://linuxmonk.ch/) | [e-mail](mailto:reto.gantenbein@linuxmonk.ch) | [GitHub](https://github.com/ganto)

License: [GPLv3](https://tldrlegal.com/license/gnu-general-public-license-v3-%28gpl-3%29)

