# https on Jenkins using Let's Encrypt

Getting a standard certificate for your jenkins, using Let's Encrypt.

(Note: this is still beta, might be some kinks left)

By comparison with the self-signed, the pre-requisites are significantly stricter:
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
  - privkey.pem

### 2. Convert the Lets Encrypt certificates to jenkins format

Given:
- you've generated `cert.pem` and `privkey.pem` in `/etc/letsencrypt/live/jenkins.mydomain.com`
- you are in the directory containing `jenkins.war`

When you do:
```
sudo cp /etc/letsencrypt/live/jenkins.mydomain.com/cert.pem .
sudo cp /etc/letsencrypt/live/jenkins.mydomain.com/privkey.pem .
openssl rsa -in privkey.pem -out key.pem
```
Then:
- `key.pem` will be generated.  This is in rsa private key format

### 3. Start jenkins

Given:
- you are in the directory containing `jenkins.war`
- `cert.pem` and `key.pem` are in this directory

When you do:
```
java -jar jenkins.war  --httpsPort=8443 --httpPort=-1 --httpsCertificate=cert.pem --httpsPrivateKey=key.pem
```
Then:
- jenkins should start
- jenkins should be available on port 8443, using https, and using your Let's Encrypt certificate

