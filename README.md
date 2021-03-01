# Multi OS Ansible Demo
## Purpose

The purpose of this demo is to demonstrate some basic multi-os [Ansible](https://www.ansible.com/) playbooks. The patterns demonstrated within the playbooks can be used when establishing a [Kubernetes](https://kubernetes.io/) cluster. For example, the main node can generate a cluster join command and set that as a fact. That fact can then be read by the other nodes and executed in their respective shells in order to join that worker host to the Kubernetes cluster.

The source files referenced in this post can be found in my public repo: [Ethan-Arrowood/multi-os-ansible-demo](https://github.com/Ethan-Arrowood/multi-os-ansible-demo).

In the `src/` folder there are three files:

1. [inventory.ini](./src/inventory.ini) - is an Ansible inventory file. It defines hosts, groups, and variables. This demo uses it to group hosts based on their OS and apply specific variables to those groups.
2. [hello-world-playbook.yml](./src/hello-world-playbook.yml) - is an Ansible playbook file that demonstrates a hello-world example where each host prints "Hello, World" to their respective shell environment.
3. [fact-playbook.yml](./src/fact-playbook.yml) is another Ansible playbook file that demonstrates a slightly more complex example of sharing information between hosts using Ansible facts. It requires Python to be installed on all hosts.

## Requirements

> [Azure Virtual Machines](https://azure.microsoft.com/en-us/services/virtual-machines/) are a great way to set up this demo environment.

- Three VMs belonging to the same vnet
- A linux based VM with public network ssh access enabled (*main*)
- A linux based VM with private vnet ssh access enabled (*linux_worker_1*)
- A windows based VM with private vnet ssh access enabled (*windows_worker_1*)

  > Notice: Azure currently does not have automatic OpenSSH Server support for Windows Server VMs. Use a pre-configured instance or follow these [guides](#windows-openssh-guides) to set it up.

- The *main* linux VM (configured with public ssh access) should have the necessary ssh keys for accessing the two other VMs (*linux_worker_1* and *windows_worker_1*).
- Make sure to sync ssh `known_host` values for the main host to the worker hosts so that the hosts trust each other prior to executing the ansible playbooks.

## Demo Steps:

1. Securely copy over the `/src` files from this directory to the *main* VM using `scp`
2. SSH into the *main* linux VM and install [Ansible](https://www.ansible.com/)
  ```sh
  sudo apt-add-repository ppa:ansible/ansible
  sudo apt update
  sudo apt install ansible
  ```
3. Execute the examples from the *main* VM using:
  ```
  ansible-playbook -i ./src/inventory.ini ./src/hello-world-playbook.yml
  ansible-playbook -i ./src/inventory.ini ./src/fact-playbook.yml
  ```
  > Notice: The `fact-playbook.yml` requires Python to be installed on the Windows Server and the path to the python interpreter added to the list of `[windows:vars]` in `src/inventory.ini` as `ansible_python_interpreter`. This will most likely be set to `C:/Python39`.

## Closing

Ansible is a fantastic tool for Infrastructure automation and management. This article is based off of a project I worked on as a Microsoft Commercial Software Engineer. If this kind of work interests you, CSE is hiring across the world for a variety of roles. Visit https://aka.ms/csejobs for details.

Cover Image by [unDraw](https://undraw.co/)

## Windows OpenSSH Guides

These guides should help in setting up Windows Server 2019 with OpenSSH. Make sure you copy over the same ssh key used by Ansible in this demo. If you're following these guides in context of this demo you'll need to allow public RDP access to the Windows instance in order to enable OpenSSH.

- [OpenSSH key management - Microsoft Docs](https://docs.microsoft.com/en-us/windows-server/administration/openssh/openssh_keymanagement)
- [Getting Started with SSH on Windows Server 2019](https://www.concurrency.com/blog/february-2019/getting-started-with-ssh-on-windows-server-2019)
- [Key-based Authentication for OpenSSH on Windows - Concurrency](https://www.concurrency.com/blog/may-2019/key-based-authentication-for-openssh-on-windows)

## Personal Scratchpad

This section contains a bunch of commands I used for completing the operations listed in the demo. They were written and used on macOS 10.15.7, so they are not guaranteed to work on other platforms.

```sh
main_vm_public_ip=<insert main host ip here>

# create a local .ssh folder
mkdir .ssh

# generate an empty passphrase ssh key in the local folder
ssh-keygen -q -m PEM -t rsa -b 4096 -N '' -f ./.ssh/id_rsa <<<y 2>&1 >/dev/null

# copy the contents of the key file into the clipboard; use this key when creating the VMs in Azure Portal
pbcopy < ./.ssh/id_rsa.pub

# copy over the physical keys for ansible main-main and main-worker connections
scp -i ./.ssh/id_rsa -r ./.ssh/ azureuser@"$main_vm_public_ip":~/

# copy over the src files for the demo
scp -i ./.ssh/id_rsa -r ./src/ azureuser@"$main_vm_public_ip":~/

# ssh into main
ssh -i ./.ssh/id_rsa azureuser@"$main_vm_public_ip"

# install ansible
sudo apt-add-repository ppa:ansible/ansible
sudo apt update
sudo apt install ansible

# run the ansible-playbooks
ansible-playbook -i ./src/inventory.ini ./src/hello-world-playbook.yml
ansible-playbook -i ./src/inventory.ini ./src/fact-playbook.yml
```