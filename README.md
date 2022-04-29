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
CA Gateway REST APIs work on a mutually authenticated TLS tunnel and hence you might require to add the CA Gateway's TLS certificate issuer's public certificate to the Ansible server's trust anchors. The steps are similar to how you would add the trust anchor of any private issuing Certificate Authority. For the ease, sample commands are below. Please be aware that these commands are specific to the OS flavor and version and it is best to refer OS' own documentation for the same.

Before running the below command(s), please create a certificate bundle file in PEM format, typical order of certs is -
- Issuing CA certificate(s)
- Root CA certificate

```
sudo cp /system_path/CA-Cert-Chain.pem /etc/pki/ca-trust/source/anchors/
sudo update-ca-trust extract
```

## Configuring plugin to talk to Entrust Certificate Authority

| Parameter | Description |
| --- | --- |
| path | Absolute path and file name where the CA signed server certifacte will be stored on the Ansible host machine |
| csr | Absolute path and file name where the server's certificate signing request will be stored on the Ansible host machine |
| cagw_api_client_cert_path | CA Gateway only allows an mTLS based incoming REST API call. <br /> This is the absolute path of the client certificate in PEM format on the Ansible host machine. |
| cagw_api_client_cert_key_path | CA Gateway only allows an mTLS based incoming REST API call. <br /> This is the absolute path of the client non-protected private key format on the Ansible host machine. |
| certificate_authority_id | CA Gateway allows configuring multiple Certificate Authorities at the backend of the same CA Gateway instance. <br /> certificate_authority_id uniquely identifies a Certificate Authority (CA) which you want to use for a particular certificate issuance. <br /> Please refer to CA Gateway documentation to get the right certificate_authority_id for this request |
| certificate_profile_id | CA Gateway allows configuring multiple certificate profiles against each Certificate Authority. <br/> certificate_profile_id uniquely identifies a certificate profile which you want to use for a particular certificate issuance. <br /> Please refer to CA Gateway documentation to get the right certificate_profile_id for this request |
| request_type | Request type can be either a 'new' or 'renew' |
| enrollment_format | Can be either 'X509' or 'PKCS12' |
| dn | Subject DN for the server certificate to be issued. <br /> Typically the server's hostname is part of the DN |
| cagw_api_specification_path | Absolute path and file name where the CAGW API specifications YAML file is present. <br /> This file contains information about the CA Gateway's FQDN which you want to connect to amongst other information.  |
| connection_type | Can be either 'SM' for Entrust Certificate Authority (private TLS certs) or 'ECS' for Entrust Certificate Services (public TLS certs) |
| requester_name | For the ECS Certificate Authority only, refer ECS documentation on how ECS uses this information |
| requester_email | For the ECS Certificate Authority only, refer ECS documentation on how ECS uses this information |
| requester_phone | For the ECS Certificate Authority only, refer ECS documentation on how ECS uses this information |
| tracking_info | For the ECS Certificate Authority only, refer ECS documentation on how ECS uses this information |
| force | Force the rekey operation |
