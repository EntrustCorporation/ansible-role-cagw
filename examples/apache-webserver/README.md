# Configuring Apache Web Server TLS certificate issued using Entrust CA Gateway
This playbook will allow automating delivering TLS certificate/key pair as well as configuring an endpoint by installing Apache Webserver, certificate/key pair, as well as configuring the webserver with TLS certificate.

## Who can use these playbooks?
If you are a DevOps engineer who oversees deployment of webservers like Apache using Ansible, this is perfect for you.
This playbook will allow adding x509 certificate/key pair that can be used to add TLS capabilities to the Apache Webserver.

## Workflow
![image](https://user-images.githubusercontent.com/98990887/166394855-3f3f151a-b50e-4414-865b-50921b40d195.png)

## Prerequisites
You will need: 
- Ansible installed with Entrust CA Gateway plugin installation
- Access to Entrust CA Gateway along with one of the supported Certificate Authorities (CA)
- Client certificate and private key to talk to Entrust CA Gateway
- Target endpoint server with inventory file for the deployment of Apache Web Server, SSL module and the TLS certificate and key pair

## Outline of the playbook
This sample playbook will perform below tasks -
- Generate server's private key
- Generate Certificate Signing Request with specified subject DN
- Send signing request to the CA Gateway with your preferred CA/Profile information
- Install Apache web server and SSL module on the target machine
- Copy the server's private key and certificate on the target machine so the same can be used by the Apache web server to unable TLS based communication
- Start the services

# Steps

## Step 1: Creating variables file

## Step 2: Create playbook

## Step 3: Setup and configure endpoint with Apache Webserver

## Step 4: Execute playbook
### *Note*
This playbook may require root privileges on the target machine so it is expected to generate public/private key pair for the root user and copy the public key on the Ansible server so that a passwordless SSH can be done by the Ansible server on the target machine.
Below steps may be used to do the same.
