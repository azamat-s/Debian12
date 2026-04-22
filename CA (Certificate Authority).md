# PKI and Certificate Authority (CA) Setup

## Preparation
- Create directory structure for grading
- Initialize OpenSSL database files
- Configure certificate extensions
- Initialize Environment

### Create target directory for certificates
```shell
mkdir -p /opt/grading/ca
```
### Setup OpenSSL directory structure
```shell
mkdir -p ~/ca/demoCA/newcerts
cd ~/ca
touch demoCA/index.txt
echo 1000 > demoCA/serial
echo 1000 > demoCA/crlnumber
```
