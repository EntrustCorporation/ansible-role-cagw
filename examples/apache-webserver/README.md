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

## Step 1: Copy client CA Gateway credentials

## Step 2: Creating variables file
To be able to use the sample playbook provided, the first step is to configure the variables file i.e. ``` ./defaults/main.yml ```. Things you would need to consider are:
- Entrust CA Gateway client cert and key path
- CA ID 
- Certificate Profile ID

```
working_path: /tmp
privatekey_path: '{{ working_path }}/files/server_key.pem'
csr_path: '{{ working_path }}/files/csr.pem'
dn: /CN=example.com
dNSName: example.com
cert_path: '{{ working_path }}/files/server.pem'
request_type: new
enrollment_format: X509
ca_id: ECS
profile_id: ecs-standard
remaining_days: 30
force: false
cagw_api_client_cert_path: '{{ working_path }}/files/cert.pem'
cagw_api_client_cert_key_path: '{{ working_path }}/files/key.pem'
cagw_api_specification_path: '{{ working_path }}/files/cagw-api.yaml'
connection_type: ECS
requester_name: ansible-server
requester_email: ansible@example.com
requester_phone: 222-222-2222
tracking_info: 1234567890
```

## Step 3: Create playbook

## Step 4: Setup and configure endpoint with Apache Webserver

## Step 5: Execute playbook
### *Note*
This playbook may require root privileges on the target machine so it is expected to generate public/private key pair for the root user and copy the public key on the Ansible server so that a passwordless SSH can be done by the Ansible server on the target machine.
Below steps may be used to do the same.
