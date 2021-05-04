# Deploying GoHarbor v1.8.1 on local machine

## Background Information on GoHarbor
 - Harbor is an open source registry that secures artifacts with policies and role-based access control, ensures images are scanned and free from vulnerabilities, and signs images as trusted. Harbor, a CNCF Graduated project, delivers compliance, performance, and interoperability to help you consistently and securely manage artifacts across cloud native compute platforms like Kubernetes and Docker.
 - Ref: https://goharbor.io/


## Prerequisites:
 - OS: Ubuntu 16.04/18.04
 - Binaries: docker, docker-compose, openssl
 - GoHarbor offline installer: harbor-offline-installer-v1.8.1.tgz (Please extract and overwrite the extracted files with the ones I have accordingly based on needs, namely the Create_CA cert for https and the service file needed to automate GoHarbor startup on reboots.) 

## Certificate Generation
 - You may refer to the steps that I did by using our client(PC) as Certificate Authority. There are more detailed explainations online for topics related to certificates, especially when you want to use it to enable HTTPs protocol for other applications.
``` bash
# Generate private key. This would prompt you to enter a pass phrase and reconfirm again.**
 - $ openssl genrsa -des3 -out myCA.key 4096 # Enter pass phrase for myCA.key

# Generate root certificate (.pem) 
 - $ openssl req -x509 -new -nodes -key myCA.key -sha256 -days 3650 -out myCA.pem

# Generate a secure server key for GoHarbor."
 - $ sudo openssl genrsa -out goharbor.io.key 4096

# Generate the certificate signing request file taking reference from my configuration file req.conf
 - $ sudo openssl req -new -key goharbor.io.key -out goharbor.io.csr -config req.conf

# Create the signed certificate for GoHarbor using the our Cert Authority certificate and keys with the GoHarbor signing request"
 - $ sudo openssl x509 -req -in goharbor.io.csr -CA myCA.pem -CAkey myCA.key -CAcreateserial -out goharbor.io.crt -days 3650 -sha256
```

 - Alternatively, you may use my created certificate files and ensure that there is read permissions by user,group and all users (644). Remember to change ownership accordingly for .crt file and root ownership for .key
### Installation
To install goharbor (requires **harbor.v1.8.1.tar.gz file which is extracted from harbor-offline-installer-v1.8.1.tgz**) with addons such as support for helm chart storage(Chartmuseum), Docker image vulnerability scanner(Clair) and Docker content trust (Notary) please run the following command:
 - $ sudo ./install.sh --with-chartmuseum --with-clair --with-notary

### Notes:
 - The files/folders are mainly extracted out from harbor-offline-installer-v1.8.1.tgz file downloadable from official GoHarbor repository, with some modifications. There is an additional folder(Create_CA) which I have added which stores the necessary certificate to enable the deployment of GoHarbor with https protocol enabled. I have also included a harbor.yml service file which could be used to automate the startup of GoHarbor service on a local machine that is used to host GoHarbor service in the event of reboot. The daemon.json file is for reference purpose for comparison after docker installation, which you can replace with the existing /etc/docker/daemon.json file in the computer if you are lazy to configure accordingly. 
 - By default, if https is enabled for GoHarbor, it would automatically route http request to https instead, assuming you are using port 80 and 443 for http and https port respectively. If you are not intending to enable https for GoHarbor, no certificate is required and please comment out the https block inside harbor.yml. 

* GoHarbor repository: https://github.com/goharbor/harbor/releases
