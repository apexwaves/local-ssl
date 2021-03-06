# Local-SSL
## Local network self signed SSL certificates  
### For Raspberry Pi OS, Ubuntu and other Debian based linux operating systems running an apache server.  

This write up was created with the hopes that it will aid setting up SSL certs on devices not accessible from the internet such as private home or business raspberry pi or ubuntu web servers that are only accessed on the local network.:slightly_smiling_face:

This setup is currently working with Chrome 95 on Windows and Android.

Drawbacks: The new SSL cert must be installed on all devices that need to communicate with your local server over https. Its a fast process but could be an issue for some large organizations.  
  
Note:  
 - This doc uses an example local IP address of 192.168.1.160 , you'll need to use your own web servers IP address.  
 - OpenSSL must be installed   
 - You may need to set ownership and/or permissions for the folders/files your working in to edit your files.
   
 Check if OpenSSL is already installed  
`dpkg -s openssl`  
Install if not already installed  
`sudo apt install openssl`  
##

#### 1 
#### Navigate to /etc/ssl and create a new localcerts folder, navigate to localcerts.  
`cd /etc/ssl`  
`sudo mkdir localcerts`  
`cd localcerts`

#### 2 
#### Create new file named req-config.conf file with your correct values, CN(common name) should be your local servers ip address (example 192.168.0.3). Tip: Nano editor pastes with a right click. 
`sudo touch req-config.conf`  
`sudo nano req-config.conf`

req-config.conf  
```
[req]  
default_bits = 4096  
default_md = sha256  
distinguished_name = req_distinguished_name
x509_extensions = v3_req
prompt = no
[req_distinguished_name]
C = US
ST = NC
L = City
O = Company
OU = Division
CN = 192.168.1.160
[v3_req]
basicConstraints = CA:TRUE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names
[alt_names]
IP.1 = 192.168.1.160
```
  
#### 3  
#### Create cert using req-config.conf config file.
`sudo openssl req -new -nodes -x509 -days 365 -keyout domain.key -out domain.crt -config req-config.conf`

#### 4  
#### Edit default-ssl.conf
Navigate to /etc/apache2/sites-available and edit default-ssl.conf to use new certificate paths for .crt and .key files.  

`cd /etc/apache2/sites-available`  
`sudo nano default-ssl.conf`  

Add Lines(see example below)  
`ServerName 192.168.1.160`  
`SSLCertificateFile /etc/ssl/localcerts/domain.crt`  
`SSLCertificateKeyFile /etc/ssl/localcerts/domain.key`  



default-ssl.conf
```<IfModule mod_ssl.c>
	<VirtualHost _default_:443>
		ServerAdmin webmaster@localhost
		ServerName 192.168.1.160
		DocumentRoot /var/www/html
    
		ErrorLog ${APACHE_LOG_DIR}/error.log
		CustomLog ${APACHE_LOG_DIR}/access.log combined

		SSLEngine on
		SSLCertificateFile /etc/ssl/localcerts/domain.crt
		SSLCertificateKeyFile /etc/ssl/localcerts/domain.key	

		<FilesMatch "\.(cgi|shtml|phtml|php)$">
				SSLOptions +StdEnvVars
		</FilesMatch>
		<Directory /usr/lib/cgi-bin>
				SSLOptions +StdEnvVars
		</Directory>

	</VirtualHost>
</IfModule>
```

#### 5  

`sudo a2enmod ssl`  
`sudo a2ensite default-ssl`  

#### Restart apache  
`sudo service apache2 restart`

#### 6  
#### Export SSL cert via browser
On Chrome desktop, visit the ip address of your server via https, click the warning icon to left of URL, view certificate->details tab->save to file. Export certificate using default settings.

#### Import SSL cert via browser
 - Chrome: settings->privacy&security->security->manage certificates navigate to manage certificates -> trusted root certificate authorities tab and import your new certificate.
 - Android: Settings->Biometrics and Security->Other security settings->Install certificate from storage, install as CA certificate.

#### 7  
#### Perform a chrome restart, in top URL bar enter `chrome://restart` to quickly restart chrome and visit your webserver IP address to test your new self signed SSL certificate.
