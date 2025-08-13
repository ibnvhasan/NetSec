# Week 2 Applied Class Material

> The exercises are designed for students to finish in an individual capacity. The exercises are not designed to be completed in tutorial sessions but rather to give you some tasks and a starting point to continue and complete on your own.

---
## Lab Overview
Public key cryptography is the foundation of today’s secure communication, but it is subject to man-in-the-middle attacks when one side of communication sends its public key to the other side. The fundamental problem is that there is no easy way to verify the ownership of a public key, i.e., given a public key and its claimed owner information, how do we ensure that the public key is indeed owned by the claimed owner? The Public Key Infrastructure (PKI) is a practical solution to this problem. The learning objective of this lab is for students to gain the first-hand experience on PKI. By doing the tasks in this lab, students should be able to gain a better understanding of how PKI works, how PKI is used to protect the Web, and how Man-in-the-middle attacks can be defeated by PKI. Moreover, students will be able to understand the root of the trust in the public-key infrastructure, and what problems will arise if the root trust is broken.

---

## Lab Tasks

Open SecureCorp network configuration in GNS3 and start all nodes. We will add 2 new Ubuntu-24.04-plus-essentials nodes (Internal-Web and CA) to the Server LAN with static IPs to perform this lab. You can find the static IP configurations in the Appendix section at the end of this document. Before starting the configurations, make the /etc and /usr directories persistent on both Internal-Web and CA nodes by right-clicking the node > Configure > Advanced. Stop and start both nodes after the configurations.


### 1. Becoming a Certificate Authority (CA)

A **Certificate Authority (CA)** is a trusted entity that issues digital certificates. The digital certificate certifies the ownership of a public key by the named subject of the certificate. A number of commercial CAs are treated as root CAs; This is a list of the leading public SSL certificate authorities by market share. Users who want to get digital certificates issued by the commercial CAs need to pay those CAs. In this lab, we need to create digital certificates, but we are not going to pay any commercial CA. We will become a root CA ourselves, and then use this CA to issue certificate for others (e.g. servers). In this task, we will make ourselves a root CA, and generate a certificate for this CA. Unlike other certificates, which are usually signed by another CA, the root CA’s certificates are self-signed. Root CA’s certificates are usually pre-loaded into most operating systems, web browsers, and other software that rely on PKI. Root CA’s certificates are unconditionally trusted.

We will use `Internal-Web` container as web server and access the website using `Internal-Client-2`. The openssl configuration file is located in: `/usr/lib/ssl/openssl.cnf`. According to configuration file we need to create directories for our certificates, see `[CA_default]` part in the configuration file. Let’s make directories on the CA, open terminal (right click on the CA node and select console), and execute the following:

```bash
# move to /root directory
cd
mkdir pki
cd pki
mkdir demoCA
cd demoCA
mkdir certs
mkdir crl
mkdir newcerts
touch index.txt
echo 1000 > serial


# The correct directory structure is:
# `-- pki
#     |-- demoCA
#     |   |-- certs
#     |   |-- crl
#     |   |-- index.txt
#     |   |-- newcerts
#     |   |-- serial
```
- [x] **`demoCA`** is Where everything is kept
- [x] **`certs`** is Where the issued certs are kept
- [x] **`crl`** is Where the issued crl are kept
- [x] **`newcerts`** is the default place for new certs
- [x] **`index.txt`** is database index file
- [x] **`serial`** is the current serial number

All commands from here onward should be run from the pki directory. Run the below command to confirm your working directory is /root/pki.
```bash
pwd
```

By default OpenSSL configures itself as an internal CA which is only signing certificates for the same organisation. We need to change this configurations so that our CA can sign certificate for any domain name. Find the following section in the configuration file `/usr/lib/ssl/openssl.cnf` and change them as below.

```bash
# open a console in your RootCA machine
# you can use: 
nano /usr/lib/ssl/openssl.cnf
# then press ctrl + W and write policy_match to find it
[ policy_match ]
countryName = optional
stateOrProvinceName = optional
organizationName = optional
organizationalUnitName = optional
commonName = supplied
emailAddress = optional
```

Alright, we are ready to generate a self-signed certificate for our CA. This means that this CA is totally trusted, and its certificate will serve as the root certificate. You can run the following command to generate a self-signed certificate for the CA:
```bash
openssl req -new -x509 -keyout ca.key -out ca.crt
```

### 2. Creating a Certificate for **networksecurity.com**
Now, we become a root CA, we are ready to sign digital certificates for our customers. Our first customer is a company called `networksecurity.com`. We assume that the web server of `networksecurity.com` is `Internal-Web` node in Server LAN. Perform the following three steps in **`Internal-Web`** node.

#### a) Generate public/private key pair:
The company needs to first create its own public/private key pair. We can run the following command to generate an RSA key pair (both private and public keys). The keys will be stored in the file server.pem:
```bash
cd
apt install openssl
openssl genrsa -out server.pem 2048
```
The `server.pem` is an encoded text file, so you will not be able to see the actual content, such as the modulus, private exponents, etc. To see those, you can run the following command: 
```bash
openssl rsa -in server.pem -text
```


#### b) Generate a Certificate Signing Request (CSR):
Once the company has the key file, it should generate a Certificate Signing Request (CSR), which basically includes the company’s public key. The CSR will be sent to the CA, who will generate a certificate for the key (usually after ensuring that identity information in the CSR matches with the server’s true identity). Please use `networksecurity.com` as the common name of the certificate request.
```bash
openssl req -new -key server.pem -out server.csr
```


#### c) Generating Certificates:
The CSR file needs to have the CA’s signature to form a certificate. In the real world, the CSR files are usually sent to a trusted CA for their signature. In this lab, we will use our own trusted CA to generate certificates. Copy the content of server.csr from `Internal-Web` to CA (refer to [Appendix 1](#b1-option-1)). Execute the following command on CA to turn the certificate signing request `server.csr` into an X509 certificate server.crt, using the CA’s `ca.crt` and `ca.key`:
```bash
openssl ca -in server.csr -out server.crt -cert ca.crt -keyfile ca.key
```
Copy the certificate file (`server.crt`) to the `Internal-Web` server (refer to [Appendix 1](#b1-option-1) again)


### 3. Deploying Certificate in an HTTPS Web Server (Apache)

Perform the following on `Internal-Web` node.

- Install Apache server and a text editor (nano):
```bash
apt update
apt install apache2
```
> If you get a Perl base dependency error when installing Apache2, run the following command to install the specific perl-base version (Optional).
> `apt install perl-base=5.38.2-3.2build2`

An Apache server can simultaneously host multiple websites. It needs to know the directory where a website’s files are stored. This is done via its VirtualHost file, located in the `/etc/apache2/sites-available` directory. To add an HTTP website, we add a VirtualHost entry to the file `000-default.conf` (using `nano /etc/apache2/sites-available/000-default.conf`). Add the following lines at the end of the file:
```bash
nano /etc/apache2/sites-available/000-default.conf

# Then add these following lines:
# <VirtualHost *:80>
    #...
    ServerName networksecurity.com
    DocumentRoot /var/www/networksecurity
    DirectoryIndex index.html
    #...
# </VirtualHost>
```
and to the file `nano /etc/apache2/sites-available/default-ssl.conf`:
```bash
nano /etc/apache2/sites-available/default-ssl.conf

# Then add these following lines: 
# <VirtualHost *:443>
    # ...
    ServerName networksecurity.com
    DocumentRoot /var/www/networksecurity
    DirectoryIndex index.html
    SSLEngine On
    SSLCertificateFile /etc/apache2/server.crt
    SSLCertificateKeyFile /etc/apache2/server.pem
    # ...
# </VirtualHost>
```

- The `ServerName` entry specifies the name of the website, 
- while the `DocumentRoot` entry specifies where the files for the website are stored. The above example sets up the HTTPS site https://networksecurity.com (port 443 is the default HTTPS port). 
- In the setup, we need to tell Apache where the server certificate (`/etc/apache2/server.crt`) and private key (`/etc/apache2/server.pem`) are stored.

First, we will create our `DocumentRoot` directory and copy our certificate and private key files in apache directory. Then, we will run few commands to test our apache configuration and finally we will restart our apache server:
```bash
mkdir /var/www/networksecurity
cp server.crt /etc/apache2/server.crt
cp server.pem /etc/apache2/server.pem
echo 'ServerName 127.0.0.1'>> /etc/apache2/apache2.conf
apachectl configtest
a2enmod ssl
a2ensite default-ssl
service apache2 restart
```

Let us also create one test html file:
```bash
nano /var/www/networksecurity/index.html
```
```html
<!-- Copy the following in html file and press ctrl+x to close and save. -->
<!DOCTYPE html>
<html>
    <body>
        <h1>Network Security</h1>
        <p>This is a test page.</p>
    </body>
</html>
```

#### 4. Browsing `networksecurity.com`

We can use one of our clients to visit the newly created HTTPS website. Open on `Internal-Client-2` which you added in the week 1 and edit the `/etc/hosts` file. We point the URL `neworksecurity.com` to our server (If you don’t have `Internal-Client-2`, add a new node to teh Corp LAN. 
> **Do not use the exiting Internal-Client or Internal-Attacker nodes)**
```bash
nano /etc/hosts

# And add the following line at the end of file:
10.10.5.80 networksecurity.com

# Press ctrl+x to save and exit. We will use text-based browser lynx to browse the website:

apt install lynx
lynx https://networksecurity.com
```

> **Question**: Why `lynx` is complaining about the certificate? Press `y` to accept the risk. You should see your webpage (index.html).

> **Answer**: Because Internal-Client-2 default OS hasn't listed the domain `networksecurity.com` in their trusted list. We must put the certificate manually. However, instead of putting server's certificate, we can put the RootCA's certificate, so every certificate signed by `RootCA` will be trusted by `Internal-Client-2`. Much more efficient than manually add every service's certificate onwards.

> Had our certificate been assigned by a public CA such as GlobalSign or Sectigo, we will not have such an error message, because Public CA certificate is very likely pre-loaded into lynx’s certificate repository already. Unfortunately, the certificate of networksecurity.com is signed by our own CA (i.e., using ca.crt), and this CA is not recognized by lynx.

We can include our CA certificate in lynx. In CA server, copy the content of ca.crt:
```bash
cat ca.crt

# Select the contents and copy it. In `Internal-Client-2`, open ca-certificates file:
apt install ca-certificates
nano /etc/ssl/certs/ca-certificates.crt
```

Paste contents of certificate, which you copied from server, at the top of `ca-certificates.crt` file. Now again `lynx` to `networksecurity.com`, and this time it should not complain about the certificate.
```bash
lynx https://networksecurity.com
```

---

## Acknowledgements
Parts of this lab is based on the SEED project (Developing Instructional Laboratory for Computer Security Education) 
> Read more: [SeedSecurityLabs](https://seedsecuritylabs.org)

### A Appendix: IP Configuration
#### IP configuration for Internal-Web Server
```bash
#
# This is a sample network config, please uncomment lines to configure the network
#

# Uncomment this line to load custom interface files
# source /etc/network/interfaces.d/*

# Static config for eth0
auto eth0
iface eth0 inet static
	address 10.10.5.80
	netmask 255.255.255.0
	gateway 10.10.5.1
	up echo nameserver 8.8.8.8 > /etc/resolv.conf

# DHCP config for eth0
#auto eth0
#iface eth0 inet dhcp
hostname Internal-Web
```

#### IP configuration for CA Server
```bash
#
# This is a sample network config, please uncomment lines to configure the network
#

# Uncomment this line to load custom interface files
# source /etc/network/interfaces.d/*

# Static config for eth0
auto eth0
iface eth0 inet static
	address 10.10.5.100
	netmask 255.255.255.0
	gateway 10.10.5.1
	up echo nameserver 8.8.8.8 > /etc/resolv.conf

# DHCP config for eth0
#auto eth0
#iface eth0 inet dhcp
hostname CA
```

### B Copying files between nodes

#### B.1 Option 1
The files used in this lab (certificate files, key files and CSR files) are base-64 encoded. They can be opened using the text editor nano or cat command. To copy files, follow the below steps.

```bash
# 1. Open the file using nano on the source node.
nano server.crt

# 2. Select the content using your mouse > right-click > Copy.

# 3. Create a new file on the destination node using nano editor with the same name.

# 4. Open the file using nano on the source node.
nano server.crt

# 5. Right-click > Paste the content.

# 6. Press Control + X to save the content as the new file.
```

#### B.2 Option 2
To copy files using SCP follow the below steps.
```bash
# 1. Start SSH service on the Internal-Web server.
nano /etc/ssh/sshd_config
# edit these following lines
    PermitRootLogin yes
    PasswordAuthentication yes

# 2. Add the following lines to the SSH configuration file on the Internal-Web server.
service ssh restart

# 3. Set a password for the root user on Internal-Web server. You can provide a simple password here.
passwd

# 4. Copy the file from the CA server to the /etc/apache directory in the Internal-Web server.
scp server.crt root@10.10.5.80:/etc/apache2
```

#### B.3 Option 3
To copy files using Netcat.
```bash
# 1. install Netcat on both CA and Internal-Server.
apt update
apt install netcat-openbsd

# 2. Run the below command on the Internal-Web server to listen to the file sent from CA.
nc -l -p 9000 > server.crt

# 3. Run the below command on the CA server to send the file to the listening port 9000 of Internal-Web server.
nc 10.10.5.80 9000 < ca.crt
```