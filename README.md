# RHEL 9 Air-Gapped Lab: Vagrant & Ansible

A professional-grade local infrastructure laboratory designed to simulate an air-gapped production environment. It automates the deployment of a Red Hat Enterprise Linux 9 cluster using Vagrant, injecting a local RHEL ISO to serve as a centralized HTTP package repository for worker nodes via Ansible.

## Architecture

The lab consists of 5 virtual machines provisioned automatically via VirtualBox:

| Node | Hostname | IP Address | Role |
| :--- | :--- | :--- | :--- |
| Control | control.ansible.lab | 192.168.56.10 | Ansible Controller & HTTP Repository Mirror |
| Worker 1-4 | nodo[1-4].ansible.lab | 192.168.56.11 - .14 | Target nodes configured for offline package consumption |

## Technical Highlights (SRE / DevOps Approach)

- Dynamic Provisioning (DRY): Worker nodes are provisioned using Ruby loops in the Vagrantfile to avoid code duplication.
- Secrets Management: Red Hat credentials are never hardcoded. They are passed securely via system environment variables.
- Hardware Automation: The RHEL 9 ISO is automatically mounted into the control node's virtual CD/DVD drive via vb.customize VBoxManage commands.
- Dynamic Inventory: The control node automatically generates its own Ansible inventory.ini using Bash scripting during the provisioning phase.
- Air-Gapped Simulation: A multi-play Ansible playbook configures the control node as an Apache (httpd) server to mirror the ISO packages, and isolates the worker nodes to exclusively consume this local mirror without internet access.

## Prerequisites

- VirtualBox
- Vagrant
- A Red Hat Enterprise Linux 9 ISO file (.iso).

## Quick Start

1. Clone the repository:
    git clone https://github.com/oriol-devops/RHCE-lab.git
    cd RHCE-lab

2. Export your Red Hat Developer credentials (required to register the control node and download the initial EPEL/Ansible dependencies):
    export RHEL_USER="your_username"
    export RHEL_PASS="your_password"

3. Launch the infrastructure:
    vagrant up

    Note: If you haven't hardcoded the iso_path in the Vagrantfile yet, the script will pause and prompt you to do it.

4. Execute the Ansible Playbook (Once Vagrant finishes provisioning, access the control node and run the automated configuration):
    vagrant ssh control
    ansible-playbook -i inventory.ini /vagrant/iso_config.yml

## Teardown

To destroy the laboratory and free up resources:
    vagrant destroy -f