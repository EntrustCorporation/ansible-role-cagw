*Draft Mode*

# Introduction
This repository contains sample playbooks that can be leveraged as-is or with minor modifications/customizations along with Entrust PKI stack, specifically CA Gateway and one or more backend Entrust Certificate Authorities.

# Setup
This section will walk you through the process of setting up the CA Gateway Ansible module and configure it to work with Entrust CA Gateway.

*Assuming the host OS is a Centos/RHEL 8 machine*

## Installing Ansible
Ansible installation steps may change as per the host Operating System or version of Operating System.
Below steps are specific to the OS we have validated with. 
For better guidance refer Ansible's own documentation.
```
# Update Cache
sudo dnf makecache
# Install epel release
sudo dnf install epel-release
# Update cache again
sudo dnf makecache
# Now install Ansible
sudo dnf install ansible
# Verify installation and version
ansible --version
```

## Getting Entrust CA Gateway Ansible plugin

## Installing plugin

## Configuring plugin to talk to Entrust Certificate Authority
