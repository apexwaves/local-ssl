# Local-SSL
## Overview: Local network self signed SSL certificates

### 1
Navigate to /etc/ssl and create a new localcerts folder, navigate to localcerts.  
`cd /etc/ssl`  
`sudo mkdir localcerts`  
`cd localcerts`

### 2 
Create new file named req-config.conf file with your correct values, common name should be your local servers ip address (example 192.168.0.3).
`sudo touch req-config.conf`  
```[req]  
default_bits = 4096  
default_md = sha256  
distinguished_name = req_distinguished_name
x509_extensions = v3_req
prompt = no
[req_distinguished_name]
C = US
ST = VA
L = SomeCity
O = MyCompany
OU = MyDivision
CN = 192.168.1.160
[v3_req]
basicConstraints = CA:TRUE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names
[alt_names]
IP.1 = 192.168.1.160```

### 3
Run `sudo openssl req -new -nodes -x509 -days 365 -keyout domain.key -out domain.crt -config req-config.conf`

### 4
Navigate to /etc/apache2/sites-available and edit default-ssl.conf to use new certificate paths for .csr and .key files
Add `ServerName 192.168.0.3` near the top your default-ssl.conf

### 5
Restart apache `sudo service apache2 restart`

### 6
On Chrome desktop, visit the ip address of your secure via https, click the warning icon to left of URL and export certificate using default settings.
In chrome security settings, navigate to manage certificates -> trust root certificates tab and import your new certificate

### 7
Perform a chrome restart `chrome://restart` to test your new valid self signed SSL certificate
