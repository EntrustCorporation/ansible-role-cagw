# Configuring F5 BIG IP Clint SSL profile for SSL offloading with Entrust CA Gateway Ansible Role
This playbook will allow automating delivering TLS certificate/key pair as well as configuring client SSL profile associated with a Virtual IP within F5 BIG IP platform. The end result will provide TLS/SSL certificates at the F5 Load Balancer and allow SSL offloading for traffic destined to endpoints like Apache and NGINX webservers.

## Prerequisites
You will need: 
- Ansible installed with Entrust CA Gateway plugin installation
- Access to Entrust CA Gateway along with one of the supported Certificate Authorities (CA)
- Client certificate and private key to talk to Entrust CA Gateway
- BIG IP instance where the SSL/TLS cert and key pair need to be pushed

## Outline of the playbook
This sample playbook will perform below tasks -
- Generate server's private key
- Generate Certificate Signing Request with specified subject DN
- Send signing request to the CA Gateway with your preferred CA/Profile information
- Create and configure a client SSL profile with above certificate and key pair
- Create VIP and associate backend web servers for the same
- Assign the SSL profile to the VIP

# Steps

## Step 1: Copy client CA Gateway credentials
To be able to connect to Entrust CA Gateway REST APIs, Ansible client would need client certificate and private key.
Follow Entrust CA Gateway's documentation to get the REST API client credentials.
Copy the REST API cert and key as ```apache_server_private_key.pem``` and ```apache_server_certificate.pem```

## Step 2: Creating variables file
To be able to use the sample playbook provided, the first step is to configure the variables file i.e. ``` ./defaults/main.yml ```. Things you would need to consider are:
- ```working_path```: Working path is the root directory where all the static files are stored and where you expect generated CSR, cert, and key to be stored 
- ```cagw_api_client_cert_path``` & ```cagw_api_client_cert_key_path```: Entrust CA Gateway client cert and key path
- ```ca_id```: CA Gateway backend CA ID (refer Entrust CA Gateway guide to know more)
- ```profile_id```: CA Gateway CA Certificate Profile ID (refer Entrust CA Gateway guide to know more)
- ```enrollment_format```: X509 or PKCS12 
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
## Step 4: Copy certs and keys to the F5 BIG IP
![image](https://user-images.githubusercontent.com/98990887/166520471-51d8dc55-e62d-4d79-9ca2-fd63ad4fb321.png)
```
- name: Copy cert to F5
  f5networks.f5_modules.bigip_ssl_certificate:
    name: cagw-f5-cert-demo
    state: present
    content: "{{ lookup('file', 'server.crt') }}"
    provider:
      password: ChangeIt!
      server: REPLACE_WITH_BIGIP_FQDN
      user: BIGIP_USERNAME
      validate_certs: no
      server_port: 8443
  delegate_to: localhost
 
- name: Copy private key to F5
  f5networks.f5_modules.bigip_ssl_key:
    name: cagw-f5-private-key-demo
    state: present
    content: "{{ lookup('file', 'server.key') }}"
    provider:
      password: ChangeIt!
      server: REPLACE_WITH_BIGIP_FQDN
      user: BIGIP_USERNAME
      validate_certs: no
      server_port: 8443
  delegate_to: localhost
 
- name: Copy CA certificate chain to F5
  bigip_ssl_certificate:
    name: cagw-f5-ca-bundle-demo
    state: present
    content: "{{ lookup('file', '/tmp/ca.crt') }}"
    provider:
      password: ChangeIt!
      server: REPLACE_WITH_BIGIP_FQDN
      user: BIGIP_USERNAME
      validate_certs: no
      server_port: 8443
  delegate_to: localhost
```
## Step 5: Create Client SSL profile with above cert chain and private key
![image](https://user-images.githubusercontent.com/98990887/166520538-e8e4e89b-5f13-4237-9ccf-38fdc7fb785b.png)
```
- name: Create a client SSL profile with a CAGW issued cert/key/chain setting
  bigip_profile_client_ssl:
    state: present
    name: clientssl_cagw-demo
    cert_key_chain:
      - cert: cagw-f5-cert-demo
        key: cagw-f5-private-key-demo
        chain: cagw-f5-ca-bundle-demo
    provider:
      password: ChangeIt!
      server: REPLACE_WITH_BIGIP_FQDN
      user: BIGIP_USERNAME
      validate_certs: no
      server_port: 8443
  delegate_to: localhost
```
## Step 6: Execute playbook
```
ansible-playbook playbook.yml
```
