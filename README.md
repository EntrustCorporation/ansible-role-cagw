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
Entrust CA Gateway Ansible module is currently available only via Entrust's own software distribution channel.
You may reach out to your sales representative from Entrust to get an access.

We are working to get the module reviewed and eventually being made available via Ansible Galaxy, but meanwhile you can contact Entrust to get the module.

## Installing plugin
CA Gateway Ansible module is available as a tar.gz archive file and can be installed as any standard private module.
Once the Ansible is installed on the host machine, please follow the stwps below to install the module in your own Ansible server instance.

```
ansible-galaxy collection install community-crypto-1.0.0.tar.gz
```

## Updating Ansible host trust store to establish mTLS connection with CA Gateway

## Configuring plugin to talk to Entrust Certificate Authority

| Parameter | Description |
| --- | --- |
| path | Absolute path and file name where the CA signed server certifacte will be stored on the Ansible host machine |
| csr | Absolute path and file name where the server's certificate signing request will be stored on the Ansible host machine |
