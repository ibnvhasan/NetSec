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
Once the company has the key file, it should generate a Certificate Signing Request (CSR), which basically includes the company's public key. The CSR will be sent to the CA, who will generate a certificate for the key (usually after ensuring that identity information in the CSR matches with the server's true identity). Please use `networksecurity.com` as the common name of the certificate request.
```bash
openssl req -new -key server.pem -out server.csr
```


#### c) Generating Certificates:
The CSR file needs to have the CA's signature to form a certificate. In the real world, the CSR files are usually sent to a trusted CA for their signature. In this lab, we will use our own trusted CA to generate certificates. Copy the content of server.csr from `Internal-Web` to CA (refer to [Appendix 1](appendix.md#b1-option-1)). Execute the following command on CA to turn the certificate signing request `server.csr` into an X509 certificate server.crt, using the CA's `ca.crt` and `ca.key`:
```bash
openssl ca -in server.csr -out server.crt -cert ca.crt -keyfile ca.key
```
Copy the certificate file (`server.crt`) to the `Internal-Web` server (refer to [Appendix 1](appendix.md#b1-option-1) again)

