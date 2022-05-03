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
### *Note*
This playbook may require root privileges on the target machine so it is expected to generate public/private key pair for the root user and copy the public key on the Ansible server so that a passwordless SSH can be done by the Ansible server on the target machine.
Below steps may be used to do the same.

## Step 1: Copy client CA Gateway credentials
To be able to connect to Entrust CA Gateway REST APIs, Ansible client would need client certificate and private key.
Follow Entrust CA Gateway's documentation to get the REST API client credentials.
Copy the REST API cert and key as ```apache_server_private_key.pem``` and ```apache_server_certificate.pem```

## Step 2: Creating variables file
To be able to use the sample playbook provided, the first step is to configure the variables file i.e. ``` ./defaults/main.yml ```. Things you would need to consider are:
- working_path: Working path is the root directory where all the static files are stored and where you expect generated CSR, cert, and key to be stored 
- cagw_api_client_cert_path & cagw_api_client_cert_key_path: Entrust CA Gateway client cert and key path
- ca_id: CA Gateway backend CA ID (refer Entrust CA Gateway guide to know more)
- profile_id: CA Gateway CA Certificate Profile ID (refer Entrust CA Gateway guide to know more)
- enrollment_format: X509 or PKCS12 
```
working_path: /tmp
csr_path: '{{ working_path }}/files/csr.pem'
privatekey_path: '{{ working_path }}/files/apache_server_private_key.pem'
cert_path: '{{ working_path }}/files/apache_server_certificate.pem'
request_type: new
enrollment_format: X509

# Apache webserver hostname
dn: /CN=example.com
dNSName: example.com

# CA and certificate profile ID
ca_id: ECS
profile_id: ecs-standard

cagw_api_client_cert_path: '{{ working_path }}/files/cagw_client_cert.pem'
cagw_api_client_cert_key_path: '{{ working_path }}/files/cagw_client_key.pem'
cagw_api_specification_path: '{{ working_path }}/files/cagw-api.yaml'

# Use ECS for public SSL certificates
# Use SM for private TLS certs issued via Entrust Certificate Authority (ECA)
connection_type: ECS

# ECS specific client information
requester_name: ansible-server
requester_email: ansible@example.com
requester_phone: 222-222-2222
tracking_info: 1234567890

remaining_days: 30
force: false
```

## Step 3: Create playbook
Once the variables are defined, you can use the sample playbook as is which is there in ```./tasks/main.yml``` or modified per you need.
Below is a sample cagw_certificate task that will use a CSR and private key to send a request to Entrust CA Gateway for certificate signing operation.
For a web server like Apache, you would need to construct your subject DN or define SAN attribute like DNS Name to include the FQDN for the webserver.
```
- name: Request a certificate from CAGW
  community.crypto.cagw_certificate:
    path: '{{ cert_path }}'
    csr: '{{ csr_path }}'
    cagw_api_client_cert_path: '{{ cagw_api_client_cert_path }}'
    cagw_api_client_cert_key_path: '{{ cagw_api_client_cert_key_path }}'
    certificate_authority_id: '{{ ca_id }}'
    certificate_profile_id: '{{ profile_id }}'
    request_type: '{{ request_type }}'
    enrollment_format: '{{ enrollment_format }}'
    dn: '{{ dn }}'
    cagw_api_specification_path: '{{ cagw_api_specification_path }}'
    connection_type: '{{ connection_type }}'
    requester_name: '{{ requester_name }}'
    requester_email: '{{ requester_email }}'
    requester_phone: '{{ requester_phone }}'
    tracking_info: '{{ tracking_info }}'
    subject_alt_name:
      dNSName: '{{ dNSName }}'
    remaining_days: '{{ remaining_days }}'
    force: '{{ force }}'
```
## Step 4: Setup and configure endpoint with Apache Webserver
Below tasks are relevant for Apache webserver installation with SSL module as well as configuring the webserver to leverage the private key and the certificate generated in above tasks.
```
- name: Install the latest version of Apache
  yum:
    name:
      - httpd
      - mod_ssl
    state: latest

- name: Update httpd.conf file for apache server
  lineinfile:
    dest: /etc/httpd/conf/httpd.conf
    line: "ServerName example.com"

- name: Copy certificate file to apache server
  ansible.builtin.copy:
    src: {{ cert_path }}
    dest: '/etc/pki/tls/certs/localhost.crt'

- name: Copy private key to apache server
  ansible.builtin.copy:
    src: {{ privatekey_path }}
    dest: '/etc/pki/tls/private/localhost.key'

- name: Start service httpd, if not started
  ansible.builtin.service:
    name: httpd
    state: started
```
## Step 5: Execute playbook
```
ansible-playbook playbook.yml
```
