This project contain docker compose to run ldap and UI with TSL   

# How to generate self_signed_certificate
In this instruction i explain how to generate self-signed_certificate instead CA certificate (usually for test environment)

## 1) Generate key (in this case RSA key as Private key)
```bash
openssl genpkey -algorithm RSA -out ldap.key
```

## 2) Create 'openssl.cnf' file as configuration file for generating certificate
The most important section of this file is DNS and IP name and address   
This file is used by crt file to specify the source or owner of certificate   
so we must specify addrees of owner of certificate that checks by client to validate source  
```file
[ req ]
default_bits       = 2048
default_keyfile    = ldap.key
distinguished_name = req_distinguished_name
req_extensions     = req_ext
x509_extensions    = v3_ext

[ req_distinguished_name ]
countryName            = US
stateOrProvinceName    = California
localityName           = San Francisco
organizationName       = blubank
commonName             = ldapserver

[ req_ext ]
subjectAltName = @alt_names

[ v3_ext ]
subjectAltName = @alt_names

#CHANGE‌ THIS‌ PART‌
[ alt_names ]
DNS.1 = localhost
#DNS.2 = ldapserver  --> ldap container name
IP.1 = 127.0.0.1
#IP.2 = MACHINE‌ IP ADDRESS
```
## 3) Generate certificate from private key and openssl.cnf file
```bash
openssl req -x509 -newkey rsa:2048 -keyout ldap.key -out ldap.crt -days 365 -nodes -config openssl.cnf
```

___________

____
## Test ldap secure  connection 
### First Config certificate path for clients in ldap.conf file (/etc/ldap/ldap.conf)
```file
TLS_CACERT /container/service/slapd/assets/certs/ldap.crt
TLS_REQCERT     demand
```

## Command to test ldap secure connection:
```bash
ldapsearch -x -b "dc=txvlab,dc=local" -H ldaps://<Your_External_IP_or_Hostname>:636 -D "cn=user-ro,dc=txvlab,dc=local" -w pass123 -ZZ -d 1
```
-d 1 is for verbose ode
