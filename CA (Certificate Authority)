# PKI and Certificate Authority (CA) Setup Guide

This guide describes the process of creating a Root CA, generating a Certificate Revocation List (CRL), and issuing service certificates for `dmz.ws.kz` with AIA and CDP extensions.

## 1. Environment Preparation
Create the necessary directories and OpenSSL database files for tracking certificates.

```shell
# Destination folder for grading
mkdir -p /opt/grading/ca

# OpenSSL structure
mkdir -p ~/ca/demoCA/newcerts
cd ~/ca
touch demoCA/index.txt
echo 1000 > demoCA/serial
echo 1000 > demoCA/crlnumber
```

## 2. Root CA Generation

### Create Root CA
Generate the private key and a self-signed certificate for the **WS Root CA**.

```shell
# Generate Root Key
openssl genrsa -out root.key 4096

# Generate Root Certificate
openssl req -x509 -new -nodes -key root.key -sha256 -days 3650 \
  -subj "/CN=WS Root CA/O=WorldSkills/C=KZ" \
  -out /opt/grading/ca/ca.pem
```

### Generate Certificate Revocation List (CRL)
Required for checking revoked certificates via the CDP link.

```shell
openssl ca -gencrl -keyfile root.key -cert /opt/grading/ca/ca.pem -out /opt/grading/ca/root.crl
```

## 3. Issuing Service Certificates
We use an extension file to include **AIA (OCSP)** and **CDP (CRL link)** fields.

### Create `ext.cnf`
```ini
[web_ext]
subjectAltName = DNS:www.dmz.ws.kz
authorityInfoAccess = OCSP;URI:http://ws.kz
crlDistributionPoints = URI:http://ws.kz

[mail_ext]
subjectAltName = DNS:mail.dmz.ws.kz
authorityInfoAccess = OCSP;URI:http://ws.kz
crlDistributionPoints = URI:http://ws.kz
```

### Generate & Sign Certificates
```shell
# Web Server Certificate
openssl genrsa -out web.key 2048
openssl req -new -key web.key -subj "/CN=www.dmz.ws.kz" -out web.csr
openssl x509 -req -in web.csr -CA /opt/grading/ca/ca.pem -CAkey root.key \
  -CAcreateserial -out /opt/grading/ca/web.pem -days 365 -sha256 -extfile ext.cnf -extensions web_ext

# Mail Server Certificate
openssl genrsa -out mail.key 2048
openssl req -new -key mail.key -subj "/CN=mail.dmz.ws.kz" -out mail.csr
openssl x509 -req -in mail.csr -CA /opt/grading/ca/ca.pem -CAkey root.key \
  -CAcreateserial -out /opt/grading/ca/mail.pem -days 365 -sha256 -extfile ext.cnf -extensions mail_ext
```

## 4. HTTP Publication (Nginx)
Configure a web server to host the CA certificate and CRL at `http://ws.kz`.

### Nginx Configuration
Create `/etc/nginx/sites-available/ca`:
```nginx
server {
    listen 80;
    server_name ca.dmz.ws.kz;
    root /var/www/ca;

    location / {
        autoindex on;
    }
}
```

### Deployment
```shell
mkdir -p /var/www/ca
cp /opt/grading/ca/ca.pem /var/www/ca/
cp /opt/grading/ca/root.crl /var/www/ca/
ln -s /etc/nginx/sites-available/ca /etc/nginx/sites-enabled/
systemctl restart nginx
```

## 5. Verification
```shell
# Verify AIA and CDP fields in the certificate
openssl x509 -in /opt/grading/ca/web.pem -text -noout | grep -A 4 "Authority Information Access"

# Verify HTTP access
curl -I http://ws.kzca.pem
```
