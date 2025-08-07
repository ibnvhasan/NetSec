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
mkdir certs
mkdir crl
mkdir newcerts
touch index.txt
echo 1000 > serial
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
# you can use nano /usr/lib/ssl/openssl.cnf
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
Now, we become a root CA, we are ready to sign digital certificates for our customers. Our first customer is a company called networksecurity.com. We assume that the web server of networksecurity.com is `Internal-Web` node in Server LAN. Perform the following three steps in `Internal-Web` node.

#### a) Generate public/private key pair:
The company needs to first create its own public/private key pair. We can run the following command to generate an RSA key pair (both private and public keys). The keys will be stored in the file server.pem:
```bash
cd
apt install openssl
openssl genrsa -out server.pem 2048
```
The `server.pem` is an encoded text file, so you will not be able to see the actual content, such as the modulus, private exponents, etc. To see those, you can run the following command: 
```
openssl rsa -in server.pem -text
```

#### b) Generating Certificates:
The CSR file needs to have the CA’s signature to form a certificate. In the real world, the CSR files are usually sent to a trusted CA for their signature. In this lab, we will use our own trusted CA to generate certificates. Copy the content of server.csr from `Internal-Web` to CA. Execute the following command on CA to turn the certificate signing request `server.csr` into an X509 certificate server.crt, using the CA’s `ca.crt` and `ca.key`:
```
openssl ca -in server.csr -out server.crt -cert ca.crt -keyfile ca.key
```
Copy the certificate file (`server.crt`) to the `Internal-Web` server