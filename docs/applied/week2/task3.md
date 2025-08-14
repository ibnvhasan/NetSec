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
