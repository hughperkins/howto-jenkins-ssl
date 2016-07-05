# https on Jenkins using Let's Encrypt

Getting a standard certificate for your jenkins, using Let's Encrypt.

By comparison with the [self-signed](https://github.com/hughperkins/howto-jenkins-ssl), the pre-requisites are significantly stricter:
- you need a domain name, with an 'A' record
- you need administrative access to your server, and concretely:
  - you need to be able to serve files on ports 80 and 443
  - this is because Let's Encrypt uses control over ports 80 and 443 as evidence that you own the machine (anyone can
serve files over ports > 1024)
  - lots of discussion on this topic at https://github.com/letsencrypt/acme-spec/issues/33

## Concepts

- since Let's Encrypt installs a bunch of stuff, rather than hacking around with trying to control this, I simply do
everything from Docker.  Then I dont have to think about what things it's changing on my system.

## Procedure

### 1. Generate the certificates using Lets Encrypt

Given:
- you have Docker installed (see https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-getting-started )
- you are connected to the webserver
- webserver has internet connection
- ports 80 and 443 are available from the internet
- you have a dns A record for jenkins.mydomain.com, pointing to the webserver
- you have sudo access to the webserver

When you do:
```
git clone https://github.com/hughperkins/howto-jenkins-ssl
cd howto-jenkins-ssl/docker
sudo docker build -t letsencrypt .
sudo service apache2 stop  # if you are running apache
sudo mkdir -p /etc/letsencrypt
sudo docker run -v /etc/letsencrypt:/etc/letsencrypt -p 80:80 -p 443:443 -it letsencrypt
cd letsencrypt
./letsencrypt-auto certonly --standalone -d jenkins.mydomain.com
# (fill in email address)
# (press return twice)
exit
sudo service apache2 start  # if you stopped it earlier
```
Then:
- certificates should be generated in `/etc/letsencrypt/live/jenkins.mydomain.com`:
  - cert.pem
  - fullchain.pem
  - privkey.pem

### 2. Convert the Lets Encrypt certificates to jenkins format

Given:
- you've generated `fullchain.pem` and `privkey.pem` in `/etc/letsencrypt/live/jenkins.mydomain.com`
- you are in the directory containing `jenkins.war`

When you do:
```
sudo cp /etc/letsencrypt/live/jenkins.mydomain.com/* .
openssl rsa -in privkey.pem -out privkey-rsa.pem
```
Then:
- `privkey-rsa.pem` will be generated.  This is in rsa private key format

### 3. Start jenkins

Given:
- you are in the directory containing `jenkins.war`
- `fullchain.pem` and `privkey-rsa.pem` are in this directory

When you do:
```
java -jar jenkins.war  --httpsPort=8443 --httpPort=-1 --httpsCertificate=fullchain.pem --httpsPrivateKey=privkey-rsa.pem
```
Then:
- jenkins should start
- jenkins should be available on port 8443, using https, and using your Let's Encrypt certificate

# starting a slave

* Convert the cert.pem, from above, to cert.der:
```
 openssl x509 -outform der -in cert.pem -out cert.der
```

* create keystore, containing this cert:

```
keytool -import -alias jenins.mydomain.com -keystore cacerts -file cert.der
# reply trust certificate=yes
# put keystore password of 'changeit', or make your own password
```
* transfer this file to the slave computer somehow (eg via /var/www/html, and download from slave)
* launch slave
  * as for normal slave launch, but add `-Djavax.net.trustStore=cacerts
```
java -Djavax.net.ssl.trustStore=cacerts -jar slave.jar -jnlpUrl https://jenkins.mydomain.com:8443/computer/testnode/slave-agent.jnlp
```
=> will work ok :-)

# Connecting from your browser

- first go to https://helloworld.letsencrypt.org
  - this will add appropriate certs to your browser
- then browse to your jenkins, on https, and you should get green padlock :-)
